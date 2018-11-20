---
layout: post
title: "Starting Typescript - Part 1"
description: "How to start using TypeScript for Dynamics CE Web Resources."
thumb_image: ""
tags: [dynamics]
---

Disclaimer: I got started using [Scott Durow's](http://develop1.net/public/author/Scott) excellent [blog series](http://develop1.net/public/post/2018/06/09/Lets-start-TypeScript-Part-1) on the topic. This is just a slightly different summary of that information.

## Step 1: Create Visual Studio Project
1. Install Node.js from [https://nodejs.org/en/]. This means you can use its package manager to get type definitions.
2. Start Visual Studio and install the 'Open Command Line' extension using **Tools > Extensions and Updates**. Search for 'Open Command Line' and select **Download**.
3. Restart Visual Studio.
4. Select **New > Project** and create a new **ASP.NET Empty Web Site**.
5. Select **Add > Add New Item** and **Typescript config file** (tsconfig.json).
6. Add the compileOnSave option:
{% highlight json %}
{
  "compilerOptions": {
    "noImplicitAny": false,
    "noEmitOnError": true,
    "removeComments": false,
    "sourceMap": true,
    "target": "es5",
    "lib": [ "dom", "es6" ]
  },
  "compileOnSave": true,
  "exclude": [
    "node_modules",
    "wwwroot"
  ]
}
{% endhighlight %}
