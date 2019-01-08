---
layout: post
title: "Durable Azure Function QuickStart"
description: "Using Durable Functions to make your Azure Function perform better."
thumb_image: ""
tags: [Azure]
published: true
---

So, you have an Azure Function that does a bit of work and it's in danger of hitting the execution time limit. 
Here's how to use Durable Functions to implement a fan-out/fan-in pattern and have each Function exection run much faster.
There are, of course, other scenarios where Durable Functions makes sense and you can find out more [here](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview) and see [patterns illustrated](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-concepts).

We're going to take our poorly performing Function and split it up into three Functions. 
1. The **Client Function** which is triggered and then starts the Orchestrator Function off.
2. The **Orchestrator Function** which splits up the payload we need to process and calls the Activity Functions.
3. The **Activity Function** which processes each item in the payload.

Because the client function and orchestrator function wait without blocking while the activity function executes and multiple instances of the activity function are executed in paralell, each function doesn't take long to run at all. 

Azure Functions are priced by execution time and number of executions in the consumption plan. The execution time is priced at a lower rate than executions so Durable Functions may cost more to run than a non-durable version of the same code. The overall cost is low however, so it's not likely to have much impact.

Note, we don't need to do anything special to create the Durable Function. It's a matter of refactoring the code and bindings.

## Step 1: The Client Function
This Function serves as the entry point, so it has the same Function name as before. It gets a new **orchestration client binding** that lets the Function create and manage durable orchestrations.
{% highlight cs %}
[FunctionName("MyClientFunction")]
public static async Task RunMyOrchestration([ServiceBusTrigger("myqueue", AccessRights.Manage,
    Connection = "MyServiceBusConnection")]string myQueueItem,
    [OrchestrationClient]DurableOrchestrationClient starter,
    ILogger log)
{
    log.LogInformation($"Started MyClientFunction;C# Queue trigger function received: {myQueueItem}");

    // Retrieve records to process (not shown)

    var orchestrationInput = SerialiseAndCompressOrchestrationInput(input);

    var instanceId = await starter.StartNewAsync("MyOrchestration", 
      orchestrationInput);
}
{% endhighlight %}

The Durable Function uses Azure Table Storage and Azure Storage Queues to provide reliable execution of Durable Functions so the input (message) supplied to the orchestrator function can be a maximum of 64 KB. Hence the use of the SerialiseAndCompressOrchestrationInput method which looks like this:

{% highlight cs %}
private static string SerialiseAndCompressOrchestrationInput(MyOrchestrationInput input)
{
    var inputString = JsonConvert.SerializeObject(input);
    var compressedInputString = StringCompressor.CompressString(inputString);

    return compressedInputString;
}

public static string CompressString(string text)
{
    byte[] buffer = Encoding.UTF8.GetBytes(text);
    var memoryStream = new MemoryStream();
    using (var stream = new GZipStream(memoryStream, CompressionMode.Compress, true))
    {
        stream.Write(buffer, 0, buffer.Length);
    }

    memoryStream.Position = 0;

    var compressedData = new byte[memoryStream.Length];
    memoryStream.Read(compressedData, 0, compressedData.Length);

    var zipBuffer = new byte[compressedData.Length + 4];
    Buffer.BlockCopy(compressedData, 0, zipBuffer, 4, compressedData.Length);
    Buffer.BlockCopy(BitConverter.GetBytes(buffer.Length), 0, zipBuffer, 0, 4);

    return Convert.ToBase64String(zipBuffer);
}
{% endhighlight %}

## Step 2: The Orchestrator Function
This Function controls how the payload is processed. It needs to be triggered by the client function and an **orchestration trigger**. The Durable Task Framework checkpoints the execution state of the function into table storage at any and each await statement. The function is run from its beginning again each time it is resumed from an await and the framework replays the output of activity function calls if they have already taken place. Our orchestrator function looks like this:
{% highlight cs %}
[FunctionName("MyOrchestration")]
[return: SendGrid(ApiKey = "SendGridKey")] // Output binding (since async method cannot have out parameter)
public static async Task<Mail> OrchestrateThings([OrchestrationTrigger]DurableOrchestrationContext context,
    ILogger log)
{
    var compressedInput = context.GetInput<string>();
    var input = StringCompressor.DecompressString(compressedInput);
    var orchestrationInput = JsonConvert.DeserializeObject<MyOrchestrationInput>(input);
    var recordIds = orchestrationInput.RecordIds.ToList();
    var tasks = new List<Task<MyActivityResult>>();

    foreach(var recordId in recordIds)
    {
        try
        {
            tasks.Add(context.CallActivityAsync<MyActivityResult>(
                "ProcessActivity", 
                new MyActivityInput
                {
                    RecordId = recordId,
                    etc
                });
        }
        catch (FunctionFailedException ex)
        {
            log.LogError(ex.InnerException.Message);
        }
    }

    await Task.WhenAll(tasks);

    var failedTasks = (from t in tasks where !t.Result.Success select t).ToList();

    if (failedTasks.Count > 0)
    {
        log.LogError($"{failedTasks.Count} tasks failed.");
        var mail = new Mail();
        var personalization = new Personalization();
        personalization.AddTo(new Email(ConfigurationManager.AppSettings["AlertEmailTo"]));
        mail.AddPersonalization(personalization);
        var messageContent = new Content("text/html", "Bugger, something went wrong.");
        mail.AddContent(messageContent);
        mail.Subject = "MyOrchestration Failure";
        mail.From = new Email("no-reply@mycompany.co.nz");

        return mail;
    }
    else
    {
        return null;
    }
}
{% endhighlight %}

If you ignore the error handling and the email alert, there's not much going on here (on the surface). We just initiate the activity function for each record to process and then check for any failures once we've waited for all the activity functions to execute.

Constraints on how the ochestrator function are implemented are detailed [here](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-checkpointing-and-replay). The orchestrator function needs to produce the same result each time it's replayed, so it should use the [DurableOrchestrationContext](https://azure.github.io/azure-functions-durable-extension/api/Microsoft.Azure.WebJobs.DurableOrchestrationContext.html) to get time constrained values and initiate remote operations.

We can supply [RetryOptions](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-error-handling#automatic-retry-on-failure) when the activity function is initiated and have the runtime automatically retry activity function executions that throw an exception. In practice I found this didn't work very well because if the retries were exhausted then the activity function exception caused the orchestrator function to stop after the Task.WhenAll() line. Besides the functions are executing quickly and errors probably won't be that transient so I found it better to just handle the exception in the activity function.

Note that the instance id of the orchestrator function is automatically passed back to the client function. We can't send results back so the orchestrator function does all the work. You can break it up into [sub-orchestrations](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-sub-orchestrations) though.

Here's the DecompressString method:
{% highlight cs %}
public static string DecompressString(string compressedText)
{
    byte[] zipBuffer = Convert.FromBase64String(compressedText);
    using (var memoryStream = new MemoryStream())
    {
        int dataLength = BitConverter.ToInt32(zipBuffer, 0);
        memoryStream.Write(zipBuffer, 4, zipBuffer.Length - 4);

        var buffer = new byte[dataLength];

        memoryStream.Position = 0;
        using (var stream = new GZipStream(memoryStream, CompressionMode.Decompress))
        {
            stream.Read(buffer, 0, buffer.Length);
        }

        return Encoding.UTF8.GetString(buffer);
    }
}
{% endhighlight %}

## Step 3: The Activity Function
And so to the activity function or the basic unit of work. We don't need to worry about restrictions on what we can do here. For example:
{% highlight cs %}
[FunctionName("ProcessActivity")]
public static MyActivityResult Run([ActivityTrigger]DurableActivityContext context,
    ILogger log)
{
    var input = context.GetInput<MyActivityInput>();

    try
    {
        // Process the record i.e. update CRM

        return new MyActivityResult { Success =true };
    }
    catch(Exception ex)
    {
        log.LogError(ex.Message);

        return new MyActivityResult { Success = false,
            ErrorMessage = ex.Message };
    }
}
{% endhighlight %}

Publish that lot and you'll have a Function (or three) that does the same work as the previous non-durable one.
