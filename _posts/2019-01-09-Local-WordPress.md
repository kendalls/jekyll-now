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

You should really skip the first two steps below. They're just there to remind me not to try using MariaDB next time I do this.

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

## Step Four: Configure Apache
See [/OSX-Apache-Development-Environment/](/OSX-Apache-Development-Environment/).

## Step Five: Install WordPress
1. Create a new database for WordPress to use, named something like localhost_websitename.
2. If you want to use the command line, then *brew install wp-cli*, followed by *mkdir ~/Sites/websitename && cd ~/Sites/websitename && wp core download && wp config create --dbname=localhost_websitename --dbuser=root --dbpass= --dbhost=127.0.0.1*.
3. Or, download [WordPress](https://en-nz.wordpress.org/download) and unzip it into your site folder, and then edit *wp-config-sample.php* to supply the database connection details and save it as *wp-config.php*.
4. Browse to http://yoursitename.localhost/wp-admin/install.php to finish configuring the WordPress site.

Note that you can use an *.htaccess* file to implement redirects and security restriction for the site.

Note also that WordPress won't connect to your database if MySQL (v8) uses **Strong Password Encryption** and it's probably better to use 127.0.0.1 as the database server name instead of localhost. One way to switch back to the earlier authentication method is to use the **Initialize Database** button in the MySQL Preference Pane (in System Preferences).
