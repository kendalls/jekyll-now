---
layout: post
title: "Dev Ops Works Item Management with Excel"
description: "How to bulk edit Dev Ops work items using Excel"
thumb_image: ""
tags: [Dev Ops]
published: true
---

## Prerequisites
* Office Excel 2010 or later 
* Either Visual Studio 2013 (or later) or [Team Foundation Service Standalone Office Integration (free)](https://go.microsoft.com/fwlink/?LinkId=832491&clcid=0x409)
* Be a member, typically a contributor, of the project team in Dev Ops

## Help, I don't see the Team ribbon in Excel
Check the Team Foundation Add-in is enabled:
1. Choose **Options** from the Excel **File** menu
2. Choose **Add-ins** from the left-hand navigation bar
3. Choose **COM Add-ins** from the **Manage** picklist at the bottom of the dialog, and click **Go**
4. Tick **Team Foundation Add-in**
5. You should see the Team ribbon now. Restart Excel to verify it remains.

If the Team ribbon doesn't appear at next launch:
1. Run **Regedit**
2. Find the **TFCOfficeShim.Connect.[version]** folder in any of the following paths. Choose the highest version number if there are multiple folders.
* HKCU\Software\Microsoft\Office\Excel\Addins
* HKLM\Software\Microsoft\Office\Excel\Addins
* HKLM\Software\WOW6432Node\Microsoft\Office\Excel\Addins
3. Set the value of the **LoadBehavior** key to 3
4. Click **OK** and restart Excel.

## Modify Work Items
1. Start Excel with a blank worksheet.
2. Click the **Team** tab and choose **New List**
3. Connect to the appropriate Dev Ops server and project
4. From the **New List** dialog, choose **Query List** and select a shared query.
5. Edit the work items and click the **Publish** ribbon button to save them back to the server.

It's a good habit to get into to click the **Refresh** button whenever you open the worksheet to ensure you have the latest items from the server.

## Working with Items
I find it's generally a good idea to work with a tree view to make it easy to visualise the relationships between items.

Click the **Add Child** button to add a task to a user story (for example). You need to use the appropriate title column in the worksheet depending on the level of the work item.

Click the **Choose Columns** button to modify the columns you see in the worksheet. Add the Description colum for example.

Note you can click the **Open in Web Access** button to edit any item in the Dev Ops web client.
