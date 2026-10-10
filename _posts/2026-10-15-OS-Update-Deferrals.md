---
title: "Not today, macOS: Demystifying OS update deferrals across Apple platforms"
date: 2026-10-15 09:00:00 +0100
description: "An in-depth look at OS update deferrals across iOS, iPadOS, and macOS. Exploring how deferral windows behave to help you navigate nuances and maintain fleet compliance."
categories: [Mac Management, Patching]
tags: [Jamf, macOS, DDM, Blueprints, App Privacy, OS Updates, iOS/iPadOS]
---

When Apple's major OS release comes around, the [Mac Admins](https://www.macadmins.org/){:target="_blank"} Slack inevitably lights up with conversations from admins questioning why devices are seeing OS update versions they expected would be hidden - or conversely, why a version they *hoped* would be visible is completely missing. 

Most of the time, this boils down to a misunderstanding of how deferrals actually function under the hood. 

I'm hoping this post can serve as a go-to reference resource to help clear up the confusion, navigate platform nuances, and keep your devices and users safely on the right update track.

## What are *OS Update Deferrals*?

In short, OS update deferrals are a mechanism across Apple platforms that lets admins control exactly which OS versions users see when they check Software Update.<br>
Think of it as a way to prevent your users from accidentally leaping ahead to a new shiny release before your IT stack is actually ready for it.

Apple provide the ability to defer software updates for a maximum of 90 days from the date of release.

[SOFA](https://sofa.macadmins.io/release-deferrals){:target="_blank"}, from Mac Admins Open Source has a great table for easily tracking drop dates for common deferral durations, across all of Apple's Managed Platforms.

## How do they work?

Broadly speaking, update deferrals function in the same way across all the applicable platforms (macOS, iOS, iPadOS, tvOS, and visionOS), but there are a couple of subtle differences to be mindful of.

----
Before I continue, let me explain what an OS version number means.

|OS Version|Major|Minor|Maintenance/Patch|
|:-:|:-:|:-:|:-:|
| 27.0.1|27|0|1|
| 26.7.1|26|7|1|
| 26.6.2|26|6|2

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

>As deferral of maintenance/patch updates fall under the control of minor updates on macOS, they'll be included in the minor update behaviour within this post. 
{: .prompt-info }

<!-- markdownlint-restore -->
----

Firstly, Major and Minor.

To understand the behaviour you should expect to see, you need to look at what you're expecting from the perspective of the device's *current* OS version.

A device moving *from* macOS 15.X *to* macOS 26.X, is a Major Upgrade.<br>
A device moving *from* macOS 15.X *to* macOS 27.X, is a Major Upgrade.<br>
A device moving *from* macOS 26.X *to* macOS 27.X, is also a Major Upgrade.

If the Major number of the OS version is increasing, the device will be following any specified deferrals for Major Upgrades.

## Right, how are they applied?

At the time of writing, we're in the midst of a crossover between MDM and DDM controls for a number of settings, which includes sOS update deferrals.

#### Pre-AppleOS 26.0
Deferrals were applied using the [`com.apple.applicationaccess`](https://developer.apple.com/documentation/devicemanagement/restrictions){:target="_blank"} preference domain, using a combination of the following keys: 
* enforcedSoftwareUpdateDelay
* enforcedSoftwareUpdateMajorOSDeferredInstallDelay
* enforcedSoftwareUpdateMinorOSDeferredInstallDelay
* enforcedSoftwareUpdateNonOSDeferredInstallDelay
* forceDelayedAppSoftwareUpdates
* forceDelayedMajorSoftwareUpdates
* forceDelayedSoftwareUpdates

#### AppleOS 26.X
With the AppleOS 26 releases, it was announced these keys were deprecated, to be removed in a future version of the OS.<br>
To maintain this functionality, but aligned to the - at the time - *Future of Device Management*, Apple provided a DDM replacement using the *deferrals* object in the [`com.apple.configuration.softwareupdate.settings`](https://developer.apple.com/documentation/devicemanagement/softwareupdatesettingsdeferralsobject){:target="_blank"} declaration type.

To defer Major update for 90 days, Minor updates for 30 days, and Non-OS updates for 7 days on macOS, the declaration payload would look like this:

```SoftwareUpdateSettingsDeferralsObject
{
    "Deferrals": {
        "MajorPeriodInDays": 90,
        "MinorPeriodInDays": 30,
        "SystemPeriodInDays": 7
    }
}
```

To defer updates for 45 days on non-macOS Platforms (iOS/iPadOS, tvOS, and visionOS), the declaration payload would look like this:

```SoftwareUpdateSettingsDeferralsObject
{
    "Deferrals": {
        "CombinedPeriodInDays": 45
    },
    "RecommendedCadence": "Oldest"
}
```

#### AppleOS 27 and beyond...
The forewarned removal of MDM deferrals was confirmed to be realised in Apple's release of the version 27 OS, meaning the only way now to defer updates and upgrades across the platforms is via DDM.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

>In my testing on macOS 27 and 27.0.1, the MDM profile deferrals still apply and defer successfully.<br> However, as Apple have announced this has been removed, my feeling is that it would be wrong to rely on this working as the 27 release progresses.<br>It may work now, but it may stop without warning...
{: .prompt-warning }

<!-- markdownlint-restore -->