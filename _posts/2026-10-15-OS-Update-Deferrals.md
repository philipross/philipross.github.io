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

Whilst the platforms all defer upgrades and updates in broadly the same way, there are some nuances that need to be understood.<br>

###### Firstly, let's talk about macOS.

To understand the behaviour you should expect to see, you need to look at what you're expecting from the perspective of the device's *current* OS version.

Majors
: A device moving *from* macOS 15.8 *to* macOS 26.7.1, is a major upgrade.<br>
: A device moving *from* macOS 15.8 *to* macOS 27.0.1, is a major upgrade.<br>
: A device moving *from* macOS 26.7.1 *to* macOS 27.0.1, is also a major upgrade.

If the major number of the OS version is increasing, the device will be following any specified deferrals for major upgrades.

Minors
: A device moving *from* macOs 15.8 *to* macOs 15.8.1, is a minor update.<br>
: A device moving *from* macOS 26.6 *to* macOS 26.6.2, is a minor update.<br>
: A device moving *from* macOS 26.6 *to* macOS 26.7, is also a minor update.<br>

If only the minor number is increasing, the device wil be following any minor update deferral controls.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

>macOS is the only platform that allows for major upgrade deferral period that is different to the minor update deferral period. 
{: .prompt-warning }

<!-- markdownlint-restore -->

Misunderstanding how the deferral periods work can lead to some unintended consequences, where versions you expect to be shown are hidden, or versions you expect to be hidden are shown.<br>
Let me give an example to show what that means.

Important note: the timelines in this example are ***completely fictional***, but they work for the purposes of the example so please bare with me.

Let's say you have a fleet of macOS Devices running *X* version of macOS 26.<br>
macOS 27 is released, and you have deferral periods of 90 days for majors, and 30 days for minors.<br>
Let's also imagine the following: 
* macOS 27 was released on 1st September 2026
* macOS 27.0.1 was released 30 days later, on 1st October 2026
* macOS 27.1 is released 45 days after 27.0, on 16th October 2026

Here's a timeline to help visualise that:
![Timeline depicting the example above](/assets/img/postImages/2026-10-15/1-Deferral-Example-timeline.png)

If you don't push a Software Update enforcement declaration, macOS 27 will become visible on your fleet 90 days after release - which in this example would be 30th November 2026.<br>
However, because more than 30 days have passed since macOS 27.0.1 was released, once the user had upgraded from macOS 26.X to macOS 27.0 they would then immediately be presented with the option to update to 27.0.1.

If you *did* push a Software Update enforcement declaration or had a client enrol into your DMS already running macOS 27.0, macOS 27.0.1 would be visible from 1st October.

Once the device is on 27.0, or 27.0.1, macOS 27.1 would become visible on 30th December, which is 30 days after it's release.

If a user did not upgrade from macOS 26.X to macOS 27.0 when it became visible and they were waiting for 27.0.1, that would become visible on 30th December 2026, which is 90 days after the 27.0.1 release date. Again, once they'd upgraded they'd immediately be presented with macOS 27.1 to update again.

Here's another timeline to help illustrate that:
![Timeline depicting the impact on a client based on the release schedule above](/assets/img/postImages/2026-10-15/2-Deferral-Example-impact.png)

###### Most common trip up

Most commonly, I see people asking why their devices are not seeing 27.0.1 or 27.1 available to their devices running macOS 26.X after the 90 day major deferral has lapsed.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

>This is because deferrals start from the date the OS version was released.<br>If you're expecting your device to see a major upgrade, then it will not see it until the deferral period has lapsed for that specific release. 
{: .prompt-tip }

<!-- markdownlint-restore -->


###### Now let's talk about iOS/iPadOS


iOS/iPadOS/tvOS/visionOS platforms don't differentiate between major and minor, but they do allow admins to set a `RecommendedCadence` to preference an upgrade over an update (or vice versa) should both deferrals lapse on the same date - which they often do!

## Right, how are they applied?

At the time of writing, we're in the midst of a crossover between MDM and DDM controls for a number of settings, which includes OS update deferrals.

Only macOS supports different values for major and minor deferrals. This is because 

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

To defer major update for 90 days, minor updates for 30 days, and Non-OS updates for 7 days on macOS, the declaration payload would look like this:

```json
{
    "Deferrals": {
        "MajorPeriodInDays": 90,
        "MinorPeriodInDays": 30,
        "SystemPeriodInDays": 7
    }
}
```

To defer updates for 45 days on non-macOS Platforms (iOS/iPadOS, tvOS, and visionOS), the declaration payload would look like this:

```json
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