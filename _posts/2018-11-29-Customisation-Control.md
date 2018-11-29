---
layout: post
title: "Customisation Control"
description: "How to keep D365 CE customisations under source control (with a TFS build pipeline and spkl)."
thumb_image: ""
tags: [Dynamics]
---
Note, this assumes you're using Team Foundation Server (TFS) for version control. I think there are ways to run Git commands in the build pipeline though.

## Step 1: Create project and add packages
1. Create a new class library project for the customisations (if necessary).
2. Add the **spkl** package to it. This will add the **Microsoft.CrmSdk.CoreTools** package (so spkl can use the Solution Packager).
3. You may want to add the **Open Command Line** extension as well.

## Step 2: Get initial version
1. Edit **spkl.json** in the root of the project. Specify the appropriate value for **solution_uniquename** in the **solutions** section at the bottom of the file. I used "default" to get all the customisations.
2. Open a command prompt at the project folder. You can use alt-space if you have Open Command Line installed.
3. Change directory to spkl and run **unpack.bat** to download the solution and unpack it.
4. Check in pending changes and you have a cut of the customisations under source control.

## Step 3: Create batch file
1. Add a new file, **unpack.bat**, to the root of the project. We'll use this from the build pipeline.
2. Add the following content to the file:
{% highlight bat %}
REM VSTS Build Script
@echo off
set connection=%~1
set password=%~2
set package_root=..\

REM Find spkl in the folder and sub-folders
for /R %package_root% %%G IN (spkl.exe) do (
	IF EXIST "%%G" (set spkl_path=%%G
	goto :continue)
)

:continue
@echo Deploying using %spkl_path%

REM Customisations
"%spkl_path%" unpack Build\My.Crm.Customisations\spkl.json "%connection%%password%" /p:debug

if errorlevel 1 (
	echo Error Code=%errorlevel%
	exit /b %errorlevel%
)
{% endhighlight %}
{:start="3"}
3. Replace **"My.Crm.Customisations"** with the name of your project folder.

This uses a couple of command line arguments to connect to CRM. Then it finds spkl.exe and executes the unpack command. Note that package_root and the path for spkl.json (in the spkl command) are set to provide the correct relative paths to spkl and remove any change of processing the incorrect configuration file. This will become clearer in the build definition.

## Step 4: Create the Build Pipeline
1. Go to **Azure Dev Ops** and navigate to **Pipelines > Builds**.
2. Create a new build pipeline, select **TFVC** as the source, your project in the **Server path**, and enter **Build** in the **Local path**.
3. Click **Continue**.
4. Click **Empty job** instead of a template. We only need a few steps, so we'll add them manually.
5. Now click the **Variables** tab and add a variable named **crm.connection** and **crm.password**. Set the value of the crm.connection variable to the appropriate connection string for your organisation but without the actual password in it e.g. Url=https://myorg.crm6.dynamics.com;AuthType=Office365;Username=myuser@mydomain.co.nz;Password=. Set the value of crm.password to the password for the user in the connection string and scroll across to the right and click the padlock icon to make the value secret.
6. Add a **NuGet Tool Installer** task to the build. I changed the **Version of NuGet.exe to install** to **4.4.1**.
7. Add a **NuGet task** to the build. I run a custom command, **restore Build\My.Crm.Customisations\packages.config -PackagesDirectory .\Build\packages**, since my project is part of a larger solution and we don't need all the packages for the solution.
8. Add a **Batch Script** task to the build. We want to run **$(Build.SourcesDirectory)\Build\My.Crm.Customisations\unpack.bat** with the arguments **"$(crm.connection)" "$(crm.password)"** to send our build variables in.
9. Add a **TFVC - Check-in changes** task to the build. Set **Files to check in** to the path to your project folder. Check **I understand**. Set **Recursion** to Full.
10. Click **Queue** to test the build.
6. Click the **Triggers** tab and add a **Scheduled Trigger** to run the build nightly.

The thing I like about this approach is that I just need to edit spkl.json and check it in to change which solution is unpacked. It can contain multiple solutions.

Note that we can't unpack solutions that have a cloned patch, but that's ok if we update spkl.json to target the appropriate solution when it's patched or merged.
