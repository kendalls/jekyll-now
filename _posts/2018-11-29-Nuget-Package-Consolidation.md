---
layout: post
title: "NuGet Package Consolidation"
description: "Easily ensure your projects use the same version of NuGet packages."
thumb_image: ""
tags: [Visual Studio]
---
This is so easy, I can't believe I haven't used it before.

Right-click your solution file and select **Manage NuGet Packages for Solution**.

Select a package, check the projects you want to update (or just leave them all checked as per the default), select the package version to use, and click the **Install** button.

![alt text]({{ site.url }}/images/posts/nugetpackageconsolidate.jpg "Package consolidation dialog")

Voila, all these projects use the same version of the package.
