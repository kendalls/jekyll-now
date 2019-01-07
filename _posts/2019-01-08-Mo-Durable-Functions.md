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

## Step 1: The Client Function
1. One
2. Two.
3. Three.

## Step 2: The Orchestrator Function
{% highlight cs %}
using System;
{% endhighlight %}
{:start="2"}

## Step 3: The Activity Function
