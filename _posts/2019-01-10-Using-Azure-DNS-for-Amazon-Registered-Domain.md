---
layout: post
title: "Using Azure DNS for an Amazon-registered Domain"
description: "How to use Azure DNS for a domain you registered with Amazon Route 53."
thumb_image: ""
tags: [Other]
published: true
---
To use Azure DNS to resolve names for a domain registered with Amazon Route 53 all we need to do is update the name servers for the registered domain and remove the hosted zone in Route 53.
We end up with a registered domain that looks like this:
![alt text]({{ site.url }}/images/posts/Route53.png "Registered Domain")

Once you have created a **DNS Zone** in Azure, note the four name servers that have been allocated.

Go to the AWS Console and select **Route 53** in the **Networking and Content Delivery** section. Click your registered domain and then you can edit the name servers.

You'll receive an email confirming the change has been requested. The update will be processed shortly afterwards. I didn't receive an email when the change had been processed.

You can delete the hosted zone for the domain from Route 53 once the DNS change is complete.

If you're using Github Pages then you can add an A record to the DNZ Zone in Azure. It should point to the following IP addreses:
* 185.199.108.153
* 185.199.109.153
* 185.199.110.153
* 185.199.111.153

A CNAME entry with name of www and value of yourname.github.io is a good thing as well.
