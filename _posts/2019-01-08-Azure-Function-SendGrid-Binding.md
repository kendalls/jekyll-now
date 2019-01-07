---
layout: post
title: "Sending Email from Azure Functions with SendGrid"
description: "How to utilise the SendGrid output binding to easily send email alerts from an Azure Function."
thumb_image: ""
tags: [Azure]
---

Here's all you need to do to send an email alert from your Azure Function code.

## Step 1: Create a SendGrid Account in Azure
1. Sign in to your Azure portal.
2. Click **Create a resource** in the left menu and select **SendGrid Email Delivery**.
3. Complete the form and click **Create**. You can launch the SendGrid portal from Azure (using the **Manage** button from the overview of the SendGrid account), but it's a good idea to remember the password you enter here.
4. Once the SendGrid account has been created, click on it in the portal, and then click the **Manage** button to start the email verification process.
5. From the SendGrid portal, click **Settings**, **API Keys**, and then the **Create API Key** button. Select **Full Access** for **Mail Send**. Remember the key value because it won't be displayed later.

Note, once you have verified your email, the free plan in Azure let's you send 25,000 emails/month, which is more than SendGrid by itself let's you send (100/day after the first month).

## Step 2: Add Library and Output Binding to Function
1. Add the **Microsoft.Azure.WebJobs.Extensions.SendGrid** NuGet package to your Function project. Use version 2.x for the 1.x Functions runtime and 3.x for runtime 2.x.
2. Add an **App Setting** to your Function containing the API Key value you created in the first step. I've imaginatively named it MySendGridAppSettingName in this case so it's clear that the attribute below doesn't contain the value of the key. You can (apparently) omit the ApiKey property if your app setting is named **AzureWebJobsSendGridApiKey**.
3. Add a SendGrid output binding to your function, using an out parameter or return attribute (if your Function is asynchronous). For example:
{% highlight cs %}
[FunctionName("SendEmail")]
public static void Run([SomeInputTrigger(...)] string myInput, 
  [SendGrid(ApiKey = "MySendGridKeyAppSettingName")] out Mail message) { ... }
{% endhighlight %}

{% highlight cs %}
[FunctionName("SendEmail")]
[return: SendGrid(ApiKey = "MySendGridKeyAppSettingName")]
public static async Task<Mail> Run([OrchestrationTrigger]DurableOrchestrationContext context,  ILogger log) { ... }
{% endhighlight %}

Note, the output binding doesn't show up in the generated function.json file, so don't worry about that.

## Step 3: Add Code to Send an Email
{% highlight cs %}
using SendGrid.Helpers.Mail;

var mail = new Mail();
var personalization = new Personalization();
personalization.AddTo(new Email(ConfigurationManager.AppSettings["AlertEmailTo"]));
mail.AddPersonalization(personalization);
var messageContent = new Content("text/html", "Something happened and it's not ideal.");
mail.AddContent(messageContent);
mail.Subject = "Computer says no";
mail.From = new Email("no-reply@yourdomain.co.nz");

return mail; // Or set message if using the out parameter for the binding.
{% endhighlight %}
