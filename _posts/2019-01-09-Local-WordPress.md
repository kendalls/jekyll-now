---
layout: post
title: "Local WordPress"
description: "How to setup a local WordPress environment."
thumb_image: ""
tags: [WordPress]
published: false
---
I thought I'd just register a new WordPress site to test the [AlexaCRM Integration Plugin](https://github.com/AlexaCRM/integration-dynamics).
But you can't install WordPress plugins unless you pay for the business edition. Who knew?

Alrighty then, WordPress. You give me no option but to create a local development/test environment (on OSX).

## Step One: Install MariaDB (using HomeBrew)
1. Start Terminal.
2. Run *brew doctor* to check HomeBrew is good to go. Run the indicated commands to address any warnings that are displayed.
3. Run *brew install mariadb*

At the end of the install, it helpfully displays:
* MySQL is configured to only allow connections from localhost by default
* To connect: mysql -uroot
* To have launchd start mariadb now and restart at login: brew services start mariadb
* Or, if you don't want/need a background service you can just run: mysql.server start

It looked like the current version of the MySQL Preference Pane (which can be installed with a custom install using the [MySQL install image](https://dev.mysql.com/downloads/mysql/)) would work with MariaDB, but no such luck for now because it didn't accept reconfiguring the default paths in the pane and creating symblinks didn't work either.

## Step Two: Uninstall MariaDB
There's a bit of a lack of GUI management tools for it. MySQL Workbench isn't fully compatible.
1. *brew remove mariadb*
2. *rm -rf /usr/local/val/mysql*
3. *brew cleanup*

## Step Three: Install MySQL
1. Download and run the [installation package](https://dev.mysql.com/downloads/mysql/)
2. Go to the MySQL Preferences Pane and uncheck **Start MySQL when your computer starts up** or not depending on your preference
3. Download and install [MySQL Workbench](https://dev.mysql.com/downloads/workbench/)

The MySQL install now lets you supply a password for the root user so you don't have to worry about that old palava.
