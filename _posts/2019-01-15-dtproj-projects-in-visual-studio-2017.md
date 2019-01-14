---
layout: post
title: ".dtproj Projects Unsupported in Visual Studio 2017"
description: "How to open SSIS projects in Visual Studio 2017"
thumb_image: ""
tags: [Visual Studio]
published: true
---

1. Run the **Visual Studio Installer** and check you've installed the "data storage and processing workload" you have **SQL Server Data Tools**.
2. Run the [SSDT Standalone Installer](https://docs.microsoft.com/en-us/sql/ssdt/download-sql-server-data-tools-ssdt?view=sql-server-2017#ssdt-for-vs-2017-standalone-installer) to add support for AS, IS, and RS projects.
3. Go to **Tools > Extensions and Updates** in Visual Studio to ensure that the **Microsoft Integration Services Projects** extension is enabled.
4. If you've already opened a project in Visual Studio and it failed, right-click the project and reload it.
