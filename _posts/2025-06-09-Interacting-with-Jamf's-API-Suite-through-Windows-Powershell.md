---
title: "Interacting with Jamf Pro's API suite through Windows Powershell"
date: 2025-06-09 10:00:00 +0100
description: "This introduction to the new Jamf-API-Powershell project provides a brief overview of setting up the project for your environment, and how to authenticate against your Jamf Pro instances."
categories: [API Interactions]
tags: [Jamf Pro, API, Powershell]
---

## What is the Jamf-API-Powershell Project?

The [Jamf-API-Powershell Project](https://github.com/philipross/jamf-api-powershell) is a new piece of work which I released recently.
<br> Most macOS Admins use Macs ad their daily drivers, but many also support other devices and platforms, and some organisations adopt a 'Windows-first' approach.

Whilst managing Macs requires a Mac for testing, this project aims to help Jamf Pro admins interact with the Jamf Pro API suite from a Windows client.

The Jamf Pro API suite offers powerful capabilities for interacting with Jamf Pro through command line and scripts.<br>This project is designed to leverage the APIs available for taking actions on individual clients; managing clients, or Jamf Pro configurations en masse; and for extracing information for reporting purposes.


## What can the Jamf-API-Powershell Project do?

Jamf's Developer documentation lists all of the APIs for Jamf Pro here:
- [Classic APIs](https://developer.jamf.com/jamf-pro/reference/classic-api)
- [Jamf Pro APIs](https://developer.jamf.com/jamf-pro/reference/jamf-pro-api)

If there's an endpoint on here, then the Jamf-API-Powershell Project should be able to use it.


## How do I set it up?

#### *Copying the connection file*

Once you've got a copy of the code onto your local machine, you can start running functions and commands in PowerShell.

Behind the scenes, functions within the module search for a specific file in a designated location to extract the required components for setting an OAuth header.

To create the required file at the correct location, the module includes a function called `Set-JamfConnectionFile`

![Image of Powershell session running Set-JamfConnectionFile function](/assets/img/postImages/2025-06-09/1-Set-Connection-File.png)

#### *Adding a connection array*

Once the file is in place, it needs to be populated with the information specific to your Jamf Pro environment(s).

The `Add-JamfConnectionArray` function handles this task as long as you can provide the following:
- Connection Name: A custom name that you'll use later to refer to these connection details.
- Client ID: The Client ID for the API Client setup in Jamf Pro
- Jamf Pro URL: The URL of your Jamf Pro server.


> The Client ID I've used here is obfuscated in the image above as it's just an example.

> [!IMPORTANT]
> If you're setting up connections to more than one Jamf Pro server, ensure that the Connection Name is unique for each connection.

![Image of PowerShell session running Add-JamfConnectionArray function](/assets/img/postImages/2025-06-09/2-Add-Connection-Array.png)

#### *Inputting the Client Secret for authentication*

Once the connection array is added, all the necessray components are in place and the keystone piece of data is the Client Secret.

To securely add this to the connection file, you can use the `Set-JamfConnectionSecret` function.<br>Behind the scenes, this function uses PowerShell's built-in [`Get-Credential`](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.security/get-credential?view=powershell-7.5) cmdlet to add the connection secret into the file.

![Image of PowerShell session running Set-JamfConnectionSecret function](/assets/img/postImages/2025-06-09/3-Set-Connection-Secret.png)
Shown in the image above, the PowerShell session doesn't show the Client Secret in plain text.

#### *Setting an authentication header*

That's it!<br>
You can now set an authentication header and make calls to your Jamf Pro Server.

The `Set-JamfHeader` function requires the Connection Name you set earlier to retrieve the necessary details and establish a connection.

![Image of PowerShell session running the Set-JamfHeader function](/assets/img/postImages/2025-06-09/4-Set-Header.png)


## How do I check if it's connecting successfully?


I found it easier to confirm my connection by making a simple call to get the API role with an ID of `1`.

Running `Get-JamfAPIRoleByID -roleID 1` then returns details about my API role, which verifies my connection is set successfully!


## This is really helpful, but there aren't many available actions...

This project is still in its early stages.<br>
The purpose of this post is to share the project publicly to help others start using it. I welcome contributions from other admins (see the [contributing](https://github.com/philipross/jamf-api-powershell/blob/main/CONTRIBUTING.md) details on the project GitHub).

As soon as I'm able to, I'll update the module with new functions for new capabilities.

