---
layout: post
title: "Keep Packages out of TFS"
description: "How to configure TFS and NuGet to stop adding the packages folder to source control."
thumb_image: ""
tags: [Visual Studio]
---
## Step One:
Delete your packages folder and check in the change. If it's still there then the new configuration gets ignored.

## Step Two:
Add a .tfignore file to your solution. You need to tell TFS to ignore the packages folder and the repositories.config file in it. So the file should contain:
{% highlight text %}
\packages
!\packages\repositories.config
{% endhighlight %}

Note, Windows Explorer doesn't like creating a file that starts with a fullstop and has no extension. One workaround is to add a trailing fullstop to the name (which is automatically removed).

## Step Three:
Add .nuget\NuGet.config file (and folder) to your solution. It should contain:
{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <solution>
    <add key="disableSourceControlIntegration" value="true" />
  </solution>
</configuration>
{% endhighlight %}

This stops the NuGet Package Manager mucking with source control and adding packages to your pending changes. This apparently only happens with pre-VS 2017 versions of Visual Studio. I figure it's worth creating file anyway in case someone opens the solution with an old version of Visual Studio.
