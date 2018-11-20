---
layout: post
title: "Starting Typescript - Part 1"
description: "How to start using TypeScript for Dynamics CE Web Resources."
thumb_image: ""
tags: [dynamics]
---

Disclaimer: I got started using [Scott Durow's](http://develop1.net/public/author/Scott) excellent [blog series](http://develop1.net/public/post/2018/06/09/Lets-start-TypeScript-Part-1) on the topic. This is just a slightly different summary of that information.

## Step 1: Create a Visual Studio Project
1. Install Node.js from [https://nodejs.org/en/](https://nodejs.org/en/). This means you can use its package manager to get type definitions.
2. Start Visual Studio and install the 'Open Command Line' extension using **Tools > Extensions and Updates**. Search for 'Open Command Line' and select **Download**.
3. Restart Visual Studio.
4. Select **New > Project** and create a new **ASP.NET Empty Web Site**.
5. Select **Add > Add New Item** and **Typescript config file** (tsconfig.json).
6. Add the compileOnSave option. You may as well turn on noImplicitAny (for strict type checks)
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
7. Install your favourite Web Resource deployment tool. [spkl](https://github.com/scottdurow/SparkleXrm/wiki/spkl) is one option, but that's a different topic.
8. Select the web site project in Solution Explorer and open the command line with Alt-Space. Run the following command:
{% highlight text %}
npm init
{% endhighlight %}
9. Accept all the defaults and then you should see a **packages.json** file in your project folder.
10. Run the following commands in the console window. You left it open, right?
{% highlight text %}
npm install @types/xrm --save-dev
npm install @types/jquery --save-dev
{% endhighlight %}
11. Check you have a node_modules folder in your project directory, with @types and xrm sub-folders.

## Step 2: Start writing TypeScript
1. Create a folder for your TypeScript files and add a new TypeScript File, named Fusion5.ts, to the project.
2. Add the following code to the file. We'll use this file as a standard reference in the TypeScript files we create in place of script Web Resources. This means we have Intellisense for common types and workaround the jQuery aliases being defined in the parent window to the Web Resource script.
{% highlight c# %}
/// <reference path="../node_modules/@types/jquery/index.d.ts" />
/// <reference path="../node_modules/@types/xrm/index.d.ts" />

interface JQueryWindow extends Window {
    $: JQueryStatic;
    jQuery: JQueryStatic;
}
{% endhighlight %}
3. Now, create another TypeScript file and add the following code to it. We'll use this to generate script for a Web Resource.
{% highlight c# %}
/// <reference path="Fusion5.ts" />

module Fusion5 {
    if (typeof $ === 'undefined') {
        var $ = (<JQueryWindow>parent).$;
        var jQuery = (<JQueryWindow>parent).jQuery;
    }

    export class MyEntity {
        public static OnLoad(executionContext: Xrm.Page.EventContext) {
            // Do stuff when the Form loads.
        }
    }
}
{% endhighlight %}
4. Add the .js file that is created when you save the .ts file to the project. You may also want to add the .js.map file (the source map for debugging, which we'll get to later).
5. Note, we're using the Execution Context so you'll need to enable that for the events in the Form Properties.
6. Finish implementing the Web Resource code according to your requirements. Note, you need to declare types for attributes, which looks like this:
{% highlight c# %}
var myAtttribute = formContext.getAttribute<Xrm.Attributes.OptionSetAttribute>('fus_myattribute');
var myLookupAttribute = formContext.getAttribute<Xrm.Attributes.LookupAttribute>('fus_mylookupattribute');
{% endhighlight %}
Intellisense helps work things out.

## Useful Snippets

We'll cover deployment and debugging in the next installment.
