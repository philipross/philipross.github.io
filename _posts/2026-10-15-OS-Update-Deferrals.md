---
title: "Not today, macOS: Demystifying OS update deferrals across Apple platforms"
date: 2026-10-15 09:00:00 +0100
description: "An in-depth look at OS update deferrals across iOS, iPadOS, and macOS. Exploring how deferral windows behave to help you navigate nuances and maintain fleet compliance."
categories: [Mac Management, Patching]
tags: [Jamf, macOS, DDM, Blueprints, App Privacy, OS Updates, iOS/iPadOS]
---

## What are *OS Update Deferrals*?

Update deferrals is a functionality available across Apple Platforms that allows control over the OS versions that users' can see in the Software Update section on managed devices.

Apple provide the ability to defer software updates for a maximum of 90 days from the date of release.

[SOFA](https://sofa.macadmins.io/release-deferrals){:target="_blank"}, from Mac Admins Open Source has a great table for easily tracking drop dates for common deferral durations, across all of Apple's Managed Platforms.

## Right, how are they applied?

At the time of writing, we're in the midst of a crossover between MDM and DDM controls for a number of settings, which includes sOS update deferrals.

#### Pre-AppleOS 26.0
Deferrals were applied using the [`com.apple.applicationaccess`](https://developer.apple.com/documentation/devicemanagement/restrictions){:target="_blank"} preference domain, using keys such as: 
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
    }
}
```

#### AppleOS 27 and beyond...
The forewarned removal of MDM deferrals was confirmed to be realised in Apple's release of the version 27 OS, meaning the only way now to defer updates and upgrades across the platforms is via DDM.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

>In my testing on macOS 27 and 27.0.1, the MDM profile deferrals still apply and defer successfully.<br> However, as Apple have announced this has been removed, my feeling is that it would be wrong to rely on this working as the 27 release progresses.<br>It may work now, but it may stop without warning...
{: .prompt-warning }

<!-- markdownlint-restore -->