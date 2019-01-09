---
layout: post
title: "OSX Apache Development Environment"
description: "Setting up a local development environment for Apache on OSX."
thumb_image: ""
tags: [Web Development]
published: true
---
There are many many ways to misconfigure Apache and get a 403 Forbidden error when you try to browse a site that I thought I'd better keep a note of how I got it all working.

At the time of writing this worked with Apache 2.4.34 and OSX Mojave (10.14.2).

You can check the installed version of Apache and PHP at the command line with *apachectl -v* and *php -v*.

## Step One: Reset existing configuration
Copy the content of **/etc/apache2/original** into its parent folder (apache2).

## Step Two: Create a test website
1. Create a folder named **Sites** in your user directory, if you don't already have one from an earlier version of OSX.
2. Create a folder for the test website in Sites. Let's name it **foo**.
3. Create an **index.html** file in the foo folder, with the following content:
```html
<!doctype html>
<html>
  <head>
    <title>Hello, World! | Foo</title>
  </head>
  <body>
    <h1>Hello, World!</h1>
    <p>Welcome to <strong>Foo</strong>.</p>
  </body>
</html>
```
