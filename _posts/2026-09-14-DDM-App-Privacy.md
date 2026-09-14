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

## Sounds useful! How do I do it?

Jamf have [recently released](https://learn.jamf.com/r/en-US/jamf-pro-blueprints-configuration-guide/2026-09-10){:target="_blank"} support for this control within the Blueprints UI.

However, to keep this post relevant to Mac Admins who may not use Jamf, I'll touch first on how to do this with a custom declaration, and then how that translates to the options in the Blueprints UI.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

>In all cases, it's important that your users are MDM-Enabled as this declaration is only applicable to the user scope on macOS. If your users aren't MDM-Enabled, my [previous post](https://philipross.github.io/posts/Retroactively-activating-user-channel/){:target="_blank"} gives an example on how to retroactively enable them.
{: .prompt-warning }

<!-- markdownlint-restore -->

In both cases, I'll be using the Zoom Workplace desktop app for the examples here.

To build the Declaration, in a similar way to PPPC/TCC profiles, you need to point the declaration to the relevant app by using the Bundle-ID, and the Designated Requirement.

To get the Bundle-ID of an App, run the following in Terminal:
```terminal
codesign -dv /Applications/zoom.us.app
```
And the Bundle-ID is returned with the header `Identifier`
![Terminal window showing the output of the codesign command to obtain the Bundle-ID](/assets/img/postImages/2026-09-14/2-App-Bundle-ID.png)


To get the Designated Requirement, it's the following command:<br>
```terminal
codesign -d -r - /Applications/zoom.us.app
```
The bit we need here is everything after `designated =>`
![Terminal window showing the output of the codesign command to obtain the Designated Requirement](/assets/img/postImages/2026-09-14/3-App-Designated-Requirement.png)


#### Custom Declaration

Now we've got all of the relevant information it's time to create the declaration.

The contents of my custom declaration are:

- Kind: **Configuration**
- Channel: **User**
- Type: **com.apple.configuration.app.settings**
- Payload:
```json
{
  "Privacy": {
    "PermissionDefaults": {
      "us.zoom.xos {identifier \"us.zoom.xos\" and anchor apple generic and certificate 1[field.1.2.840.113635.100.6.2.6] /* exists */ and certificate leaf[field.1.2.840.113635.100.6.1.13] /* exists */ and certificate leaf[subject.OU] = BJ4HAAB9B3}": {
        "Camera": "Allow",
        "Microphone": "Allow",
        "LocalNetwork": "Allow",
        "Accessibility": "Allow",
        "OrganizationJustification": "This app is used for Video Conferencing"
      }
    }
  }
}
```

Note that because this is JSON, it's important to escape the double quotes else the JSON object will not be valid.

![Blueprints showing the custom declaration configured](/assets/img/postImages/2026-09-14/4-Custom-Declaration.png)