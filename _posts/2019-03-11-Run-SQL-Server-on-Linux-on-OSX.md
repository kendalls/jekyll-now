---
layout: post
title: "Run SQL Server on your Mac (using Docker)"
description: "How to run SQL Server on OSX using Docker."
thumb_image: ""
tags: [OSX]
published: true
---

Install and run [Docker Community Edition for Mac](https://store.docker.com/editions/community/docker-ce-desktop-mac).
Docker has a simple interface, that appears in your menu bar. The command line is the interface to download container images and start/stop them.

Note, when you sign in to Docker use your username rather than email address otherwise you'll get an "Unauthorized: incorrect username or password" error when you try to start container images.
If you login to Docker Hub, you'll see your username on the right hand end of the menu bar.

To download the latest SQL Server image, run the following command in a Terminal window:
{% highlight %}
sudo docker pull microsoft/mssql-server-linux:2017-latest
{% endhighlight %}

To run the image, enter the following command:
{% highlight %}
sudo docker run -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=<your new password>' -p 1433:1433 --name mssql -d microsoft/mssql-server-linux:2017-latest
{% endhighlight %}
The name option is a friendly name for the container that is good to keep short because you'll use it a lot to start/stop the image and other things.

Use the following command to see the status of containers:
{% highlight %}
docker ps -a
{% endhighlight %}

The container won't automatically run when you restart your Mac, so use the following command when you need to start the container:
{% highlight %}
docker start mssql
{% endhighlight %}

Now all you need is SQL Operations Studio, the new SSMS.
