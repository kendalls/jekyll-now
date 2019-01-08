---
layout: post
title: "Durable Azure Function QuickStart"
description: "Using Durable Functions to make your Azure Function perform better."
thumb_image: ""
tags: [Azure]
published: false
---

So, you have an Azure Function that does a bit of work and it's in danger of hitting the execution time limit. 
Here's how to use Durable Functions to implement a fan-out/fan-in pattern and have each Function exection run much faster.
There are, of course, other scenarios where Durable Functions makes sense and you can find out more [here](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview).

We're going to take our poorly performing Function and split it up into three Functions. 
The **Client Function** which is triggered and then starts the Orchestrator Function off.
The **Orchestrator Function** which splits up the payload we need to process and calls the Activity Functions.
The **Activity Function** which processes each item in the payload.

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
{% highlight cs %}
using System;
{% endhighlight %}
{:start="2"}

## Step 3: The Activity Function
