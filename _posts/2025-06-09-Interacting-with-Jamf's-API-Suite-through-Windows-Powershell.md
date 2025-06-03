---
title: "Interacting with Jamf Pro's API suite through Windows Powershell"
date: 2025-06-09 10:00:00 +0100
description: "This introduction to the new Jamf-API-Powershell project provides a brief overview of setting up the project for your environment, and how to authenticate against your Jamf Pro instances."
categories: [API Interactions]
tags: [Jamf Pro, API, Powershell]
---

## What is the Jamf-API-Powershell Project?

The [Jamf-API-Powershell Project](https://github.com/philipross/jamf-api-powershell) is a new piece of work which I released recently.
<br> I know that a majority of macOS Admins use a Mac as their daily-driver, I also know that a number of admins have to support other devices and platforms, and some organisations are using a 'Windows-first' approach.

I would argue that if you're managing Macs, you definitely _need_ a Mac, at least for testing purposes, but this project is designed to help those Jamf Pro admins interact with the Jamf Pro API suite from a Windows client.

The Jamf Pro API suite is powerful and can be used to achieve a variety of things, but this project is aimed as using the APIs for management or individual clients, or clients en-masse, and for extracting information for reporting purposes.


## What can the Jamf-API-Powershell Project do?

Jamf's Developer documentation lists all of the APIs for Jamf Pro here:
- [Classic APIs](https://developer.jamf.com/jamf-pro/reference/classic-api)
- [Jamf Pro APIs](https://developer.jamf.com/jamf-pro/reference/jamf-pro-api)

If there's an endpoint on here, then the Jamf-API-Powershell Project should be able to use it.


## How do I set it up?

#### *Copying the connection file*

Once you've got a copy of the code on to your local machine, you can start running functions and commands in Powershell.

Behind the scenes, the functions are looking for a specific file in a specific location to extract the required compoments for setting an OAuth header.

To create the required file at the correct location, there's a function included in the module: `Set-JamfConnectionFile`

![Image of Powershell session running Set-JamfConnectionFile function](/assets/img/postImages/2025-06-09/1-Set-Connection-File.png)

#### *Adding a connection array*

Once you've got your file in place, it needs to be populated with the information specific to your Jamf Pro environment(s).

The `Add-JamfConnectionArray` function will take care of this, as long as you can provide the following:
- Connection Name
- Client ID
- Jamf Pro URL

The Connection Name is the custom name that you'll then use later to call these connection details. 

> The Client ID I've used here is obfuscated in the image above as it's just an example.

> [!IMPORTANT]
> If you're setting up connections to more than one Jamf Pro server, you should keep the Connection Name unique for each connection.

![Image of Powershell session running Add-JamfConnectionArray function](/assets/img/postImages/2025-06-09/2-Add-Connection-Array.png)

#### *Inputting the Client Secret for authentication*

Once the connection array is added, all the pieces are slotting into place and the keystone piece of data being the Client Secret.

To add this to the connection file securely, the function `Set-JamfConnectionSecret` can be used.<br>Behind the scenes, this adds this using Powershells built-in [Get-Credential](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.security/get-credential?view=powershell-7.5) cmdlet.

![Image of Powershell session running Set-JamfConnectionSecret function](/assets/img/postImages/2025-06-09/3-Set-Connection-Secret.png)
Shown in the image above, the Powershell session doesn't show the Client Secret in plain text.

#### *Setting an authentication header*

And that's it, you're ready to go!<br>
You can now set an authentication header and make calls to your Jamf Pro Server.

The function `Set-JamfHeader` will need the Connection Name you set earlier to pull through the required details and establish a connection.

![Image of Powershell session running the Set-JamfHeader function](/assets/img/postImages/2025-06-09/4-Set-Header.png)


## How do I check it's connecting successfully?


I thought it was easier to confirm that my connection is working with a simple call to get the API role with an ID of `1`

Running `Get-JamfAPIRoleByID -roleID 1` then returns details about my API role, which verifies my connection is set successfully!


## This is really helpful, but there's not many available actions...

This project is really only just starting.<br>
The purposes of this post is to share the project publicly to help people to start to use it. Contributions from other admins are welcomed (see the [contributing](https://github.com/philipross/jamf-api-powershell/blob/main/CONTRIBUTING.md) details on the project GitHub).

As and when I'm able to, I'll update the module with new functions for new capabilities.

