---
title: "App Privacy, Declarative Style: Reducing privacy prompt fatigue in macOS 27"
date: 2026-09-14 18:25:00 +0100
description: "Say goodbye to endless permission pop-ups with macOS 27 Golden Gate. An example of how using the new App Privacy declaration type can cut down the number of prompts seen by your users."
categories: [Mac Management, Device Configuration]
tags: [Jamf, macOS, DDM, Blueprints, App Privacy]
---

## What is *App Privacy*?

Apple's approach to privacy is well-known and well [documented](https://www.apple.com/in/privacy/control/){:target="_blank"}.<br>
For an app to have access to your Camera, Photos, Calendar, Reminders, Local Network, Location, Microphone... (the list does go on!) this permission ***must*** be expressly given by **you**.

If you've been using or managing Macs through the recent Tahoe era, you'll most likely be familiar with this pop-up:<br>
![Privacy Prompt where Google Chrome is asking permission to find devices on the local network](/assets/img/postImages/2026-09-14/1-Chrome-local-network-popup.png)

These prompts are designed to put the user in the driving seat when it comes to privacy on the device they use, but this can cause issues for Mac Admins as users may inadvertently deny a permission that's needed for devices to function seamlessly in their organisation.

macOS 27 Golden Gate takes a step forward in helping smooth that path for Mac Admins, and by extension, their users.

## How is this going to help me, or my users?

At WWDC26, Apple announced a [consolidated privacy prompt](https://www.youtube.com/watch?v=XimrZukpOfg&t=656s){:target="_blank"}. This replaces prompts for individual permissions that appear when an app, or a website accessed via Safari, try to use a feature on the device that requires user permission.

So instead of a user getting a prompt to allow access to Location Services, and then a second prompt to allow access to the Camera, and then a third prompt to allow access to the Local Network, and then a fourth prompt....*(need I go on??)*<br>
The user now gets a single prompt to allow all of the necessary permissions as defined by their organisation.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

>Important to note: the user can still deny this access.  This does not silently enable it for them; it simply guides the user to set the correct permissions, all in a single click.
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

To build the Declaration, in a similar way to PPPC/TCC profiles, you need to point the declaration to the relevant app by using the Bundle ID and the Designated Requirement.

To get the Bundle ID of an app, run the following in Terminal:
```terminal
codesign -dv /Applications/zoom.us.app
```
And the Bundle ID is returned with the header `Identifier`.
![Terminal window showing the output of the codesign command to obtain the Bundle ID](/assets/img/postImages/2026-09-14/2-App-Bundle-ID.png)



To get the Designated Requirement, run the following command:<br>
```terminal
codesign -d -r - /Applications/zoom.us.app
```
The part we need here is everything after *`designated =>`*
![Terminal window showing the output of the codesign command to obtain the Designated Requirement](/assets/img/postImages/2026-09-14/3-App-Designated-Requirement.png)


#### Custom Declaration

Now that we've got all the relevant information, it's time to create the declaration.

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

Note that because this is JSON, it's important to escape the double quotes or else the JSON object will not be valid.

![Blueprints showing the custom declaration configured](/assets/img/postImages/2026-09-14/4-Custom-Declaration.png)

We can see the Declaration lands on the client:

{%
  include embed/video.html
  src='/assets/img/postImages/2026-09-14/5-Declaration-Client-side.mp4'
  types='mov'
  title='App Privacy declaration installing on client'
  autoplay=true
  loop=true
  muted=true
%}

Next time I launch Zoom, I'm presented with the consolidated prompt to allow the services configured in my Declaration.

![Consolidated privacy prompt asking me to approve the permissions for Camera, Microphone, and Device Control and Data Access, formerly called Accessibility](/assets/img/postImages/2026-09-14/6-Consolidated-prompt.png)

I noticed that I wasn't prompted to approve Local Network access at this time, but this looks to be because Zoom hasn't included Local Network in the app entitlements, and the app hasn't yet attempted to communicate with devices on my local network.

Also note that Apple has renamed `Accessibility` to `Device Control and Data Access` within the OS.

#### Jamf Blueprints UI

Using custom declarations is great, but Jamf have made this easier by introducing native support within the Blueprints UI.

The new `App Settings` component automatically formats the payload, meaning you can bypass the manual JSON object construction altogether.<br>
If you’ve ever lost twenty minutes of your life hunting down a missing backslash or an unescaped double quote in a JSON payload, this UI will save you that headache.

![App Settings configuration in Jamf Pro Blueprints](/assets/img/postImages/2026-09-14/7-New-App-Settings-Declaration.png)

To get started, the `key` field in Blueprints is for the app's identifier - which in macOS is the composed identifier using the Bundle ID and the Designated Requirement.<br>
Because Blueprints builds the underlying JSON structure for you, you can paste the verbose output from your `codesign` commands without needing to escape quotes.

![Blueprint with the app identified added in](/assets/img/postImages/2026-09-14/8-Privacy-Declaration.png)

When selecting Configure, you're presented with drop-down controls to select the default permissions for whichever services you want to include in the declaration:

![Blueprint showing choices of the default permissions](/assets/img/postImages/2026-09-14/9-Privacy-Declaration-configure-permissions.png)

Once selected, saving the settings adds the configuration directly to your blueprint, ready to deploy across your devices.

![Blueprint showing the confirmed settings](/assets/img/postImages/2026-09-14/10-Privacy-Declaration-Configured.png)
![Showing the full configuration, ready to be added to the blueprint](/assets/img/postImages/2026-09-14/11-Blueprint-Configured.png)


## That's it!

That's all there is for configuring the new App Privacy Declaration.<br>
It might take a bit of time to get the configurations crafted, tested, and deployed to your fleet, but hopefully this short post gives you an example of what you can do with this new control if you haven't been testing it during the Betas.

### Important points about deprecations.

As a result of these controls now moving into the DDM spec, Apple have announced the deprecation for controls of `Camera`, `Microphone`, `Accessibility`, `Speech Recognition`, and `BluetoothAlways` using a PPPC/TCC configuration profile.

This doesn't mean *removed*, but it's a shot across the bow to move your controls to DDM (if you can), and that you may not get support if you encounter issues using a deprecated control.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

>`Accessibility` also has some other changes to how it works via PPPC/TCC on macOS 27.
>
>*These are detailed below.*
{: .prompt-warning }

<!-- markdownlint-restore -->

If you've got existing PPPC/TCC profiles for `Accessibility`, they will continue to enable this setting ***but*** users will see a new notification alerting them to this.

![UNC notification for accessibility prompt](/assets/img/postImages/2026-09-14/12-New-Accessibility-UNC.png)

Not only will the user get this new notification, they'll also be able to *disable* the control in System Settings, if they wanted to. 

***It is no longer greyed out.***

> *"In macOS 27.0, the device shows a non-blocking notification for each application when this setting is applied, and it allows the user to make changes to the setting in the System Settings app."*

[Source](https://github.com/apple/device-management/blob/seed_OS_27_0/mdm/profiles/com.apple.TCC.configuration-profile-policy.yaml#L166){:target="_blank"}

*And* this notification also seems to display for any app where you have a PPPC/TCC profile with `Accessibility` installed - even if the app itself isn't on the device.<br>

So if you pre-deploy PPPC/TCC profiles for apps that your users *might* install, this could be quite noisy.

<br>

<br>

##### Catch you next time!

That's all I've got today, so all that remains is for me to wish you Happy Release Day and good fortune for the journey of AppleOS 27!
