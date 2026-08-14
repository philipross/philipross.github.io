---
title: "Admin-eers, standby... 3, 2, 1... ACTIVATE: Stacking DDM predicates to lean out Jamf Blueprints"
date: 2026-08-19 08:00:00 +0100
description: "Activation Predicates are a powerful way of controlling which devices in your fleet are applied with which controls, without having to create and maintain endless blueprints and scoping groups."
categories: [Mac Management, ]
tags: [Jamf, macOS, DDM, Blueprints, Security]
---

## What's an Activation Predicate?

Whenever I hear the phrase _"Activation Predicate"_, my mind is instantly taken back to my lates 90's/early noughties childhood watching *Robot Wars* on a Friday evening.<br>

I like to think that when a Declaration lands on an Apple device, it checks for any predicate based activation conditions, and if it matches there's a tiny Craig Charles shouting *"3, 2, 1....ACTIVATE!"* and those settings are applied.

In reality, Activation Predicates aren't nearly as exhilirating as watching homemade robots knoocking seven bells out of each other, but...now that Jamf have included this ability within the Blueprints toolset, they're a really powerful way of reducing the number of blueprints you need to manage different configuration permutations across your fleets.

Apple's Deployment Guide states:

    Activations can include optional predicates that determine whether the configurations referenced in the activation are applied to the device. 
    ...
    The benefit of activation predicates is in smart use cases, where you can preload devices with declarations, which automatically activate when the device management service sends the correct management property. This approach can help avoid complex grouping and scoping on the service side.

[Link to source](https://support.apple.com/en-gb/guide/deployment/depc30268577/web#:~:text=certificates%20and%20identities.-,Activation%20predicates,-Declarative%20device%20management)

To try and demonstrate how you can use this, I'll use the example of controlling access to Beta updates to show what can be achieved with this, and how it can help reduce the number of blueprints your and your fellow admins may have to create and maintain.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> The example in this post is based on macOS, but the ability to use Activation Predicates is available on other Apple Platforms.
{: .prompt-info }
<!-- markdownlint-restore -->

### Previous configuration

With the release of AppleOS 26.0, Apple announced the deprecation of Software Update controls via a profile with the `com.apple.SoftwareUpdate` preference domain.

When using a config profile approach, it was cumbersome but understood that you could have a core set of controls included in two profiles, and then the value of the `AllowPreReleaseInstallation` key would either permit, or deny access to Beta OS Versions through the Software Update mechanisms in System Settings.

Jamf allowed for targets and exlusions so you could easily setup a smart/static group to move devices out of scope of one profile, and into scope of the other, allowing the appropriate controls to be applied to your devices.

So you'd end up with something like this:
- Profile denying access to Beta
    - Scoped to All Computers
    - Excluded from Beta tester static/smart group
- Profile permitting access to Beta
    - Scoped to Beta tester static/smart group

This worked great.<br>
All of your devices have *some* configuration applied to the Software Update settings, and you could easily allow a computer to access the Beta seeds by modifying it's group memberships in Jamf Pro.

Now though, Apple have deprecated that configuration profile, and it's been confirmed it's [*removed* in macOS 27.0](https://github.com/apple/device-management/blob/seed_OS_27_0/mdm/profiles/com.apple.SoftwareUpdate.yaml#L12)

### So...what do I we do now then?

![Screenshot from Apple's "WWDC26: What's new in managing Apple devices" YouTube video, showing DDM is not the standard for device management](/assets/img/postImages/2026-08-19/1-DDM-is-standard.png){: .left}
In line with Apple's WWDC26 statement about DDM being the <i>standard</i> for Device Management, the ability to configure Software Update settings via configuration profile will be removed, but Apple provided capability to manage these same settings via DDM with macOS Sequoia.

[Apple's Developer Docs](https://developer.apple.com/documentation/devicemanagement/softwareupdatesettings) contain more info on this.

<br>
Lorem