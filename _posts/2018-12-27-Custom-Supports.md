---
layout: post
title: "Custom Supports"
description: "Using Meshmixer to provide custom support to an object."
thumb_image: ""
tags: [3d Printing]
---
Lattice supports are good (because they work universally) but there are times where you don't want support everywhere (even if it is just from the build plate).
You might be printing multiple models that don't all need support or just need support on the outside of the object or in particular spots.

Adjusting the **Overhang Threshold** and/or **Pattern Spacing** in the **Support Material** section of the **Print Settings** tab in Slic3r can sometimes help if slicing has generated support material you don't want.

Slic3r Prusa Edition (PE) also has support (no pun intended) for **Support Blockers** and **Support Enforcers** i.e. places where you can select to have no support or places where you'd like to generate support.
However the user interface is clunky (but likely to improve in later versions of the slicer) and the rectangular blocks are hard to fit with curved shapes on models. Ask me, I know.

**To use Support Blockers/Enforcers:**
1. Double-click the model
2. Click the **Load generic** button in the settings dialog and enter the dimensions of the area you want to add/block support for.
3. Select the appropriate **Type**
4. Adjust the X/Y/Z sliders to position the box appropriately.
5. Slice the model

But there is an easier way. [Meshmixer](http://www.meshmixer.com/download.html). Which uses tree supports. And it's free.

Here's an example of where it could be useful. This is the shell of a [Volkswagen Type 2](https://fab365.net/items/95) which looks like it needs some support on the outside of the body. I say "looks like" because it turns out the underside of the lip has a 45 degree chamfer and prints ok without support, but we'll pretend it does need it to compare lattice and tree support.

![alt text]({{ site.url }}/images/posts/TransporterModel.jpg "Type 2 Shell")

The problem is that Slic3r gets carried away like so.

![alt text]({{ site.url }}/images/posts/TransporterWithLatticeSupport.jpg "Type 2 with Lattice Support")

This print will take 6 hours and 8 minutes and use 19.40 metres of filament.

However this version with tree support takes 5 hours and 31 minutes and uses 16.06 metres of filament.

![alt text]({{ site.url }}/images/posts/TransporterWithTreeSupport.jpg "Type 2 with Tree Support")

So with custom supports you get a lot less waste and a shorter print time.

**To add tree supports to the model:**
1. Start Meshmixer and click the **Import** tile
2. Click **Analysis** on the left of the screen
3. Click **Overhangs**
4. Check **Layer Height** (in the Support Generator area) matches your slicer settings
5. Click **Generate Support**
6. Control-click a tree or branch to remove a support
7. Adjust settings and click Generate Support again for more control over support generation. A lower **Angle Thresh** generates less support. As does a lower **Density**. The red areas show you where support will be generated.
8. Once you're happy, click **Export** on the left to save a new (binary) STL file.

Note, scale the model before you generate support in Meshmixer to avoid making the supports too thin or having them stick to the model excessively. Don't ask me how I know.
