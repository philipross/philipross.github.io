---
title: "App Privacy, Declarative Style: Reducing privacy prompt fatigue in macOS 27"
date: 2026-09-14 18:00:00 +0100
description: "Say goodbye to endless permission pop-ups with macOS 27 Golden Gate. An example of how using the new App Privacy declaration type can cut down the number of prompts seen by your users."
categories: [Mac Management, Device Configuration]
tags: [Jamf, macOS, DDM, Blueprints, App Privacy]
---

## What is *App Privacy*?

Apple's approach to privacy is well known, and well [documented](https://www.apple.com/in/privacy/control/){:target="_blank"}.<br>
For an App to have access to your Camera, Photos, Calendar, Reminders, Local Network, Location, Microphone... (the list does go on!) this permission ***must*** be expressely given by **you**.

If you've been using or managing Macs through the recent Tahoe era, you'll most likely be familiar with this pop-up:<br>
![Privacy Prompt where Google Chrome is asking permission to find devices on the local network](/assets/img/postImages/2026-09-14/1-Chrome-local-network-popup.png)

These prompts are designed to put the user in the driving seat when it comes to privacy on the device they use, but this can cause issues for Mac Admins as users may inadvertently deny a permission that's needed for devices to function seamlessly in their organisation.

macOS 27 Golden Gate takes a step forward in helping smooth that path for Mac Admins, and by extension, their users.

## How is this going to help me, or my users?

At WWDC26, Apple announced a [consolidated privacy prompt](https://www.youtube.com/watch?v=XimrZukpOfg&t=656s){:target="_blank"}. This replaces prompts for individual permissions that appear when an App, or a website accessed via Safari, try to use a feature on the device that requires user permission.

So instead of a user getting a prompt to allow access Location services, and then a second prompt to allow access to the Camera, and then a third prompt to allow access to the Local Network, and then a fourth prompt....<br>
The user now gets a single prompt to allow all of the necessary permissions as defined by their organisation.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

>Important to note: the user can still deny this access.  This does not silently enable it for them, it's simply guiding the user to set the correct permissions, all in a single click
{: .prompt-info }

<!-- markdownlint-restore -->

