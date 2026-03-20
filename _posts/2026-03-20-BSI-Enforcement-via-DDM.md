---
title: "Declare and Secure: Enforcing macOS Background Security Update installation using DDM"
date: 2026-03-20 14:00:00 +0100
description: "DDM Software Update plans have the capability to enforce installation of Background Security Improvements. This post will cover off the ways that this can be achieved within Jamf, working with Blueprints but also without."
categories: [Mac Management]
tags: [macOS, Jamf, Security, Patching, DDM, Blueprints]
---

## Background Security Improvewhaaa?

[Background Security Improvements](https://support.apple.com/en-gb/102657) - or BSIs - is a capability introduced into the Apple iOS, iPadOS, and macOS platforms with the 26.1 release.<br>
Prior to this release, Apple used Rapid Security Releases - RSRs - to achieve a similar goal.

This week, Apple released BSIs for the iOS, iPadOS, and macOS platforms, and it left a bit of a question on how to ensure devices across enterprise fleets installed this BSI as soon as possible.

Thankfully, the existing Declarative Device Management (DDM) Software Update (SU) mechanism that exists within the platforms can handle BSI enforcement, in the same way it can handle major, minor, or maintenance OS releases.<br>
After some trial and error, I've worked out how to enforce this for macOS using Jamf - in two methods. 
* A method using Blueprints
* A method without Blueprints

### What do the docs say?

Before I delve into _how_ to do this, I wanted to touch on what Apple's documentation says around DDM SU, and what components are required to be successful here.

Apple's GitHub page [on this topic](https://github.com/apple/device-management/blob/release/declarative/declarations/configurations/softwareupdate.enforcement.specific.yaml) has details around what's needed, and what the expected behaviour is.

For a SU declaration to be valid, it ___must___ contain two required keys:

```json
{
    "TargetOSVersion": "xx.y.z",
    "TargetLocalDateTime": "yyyy-mm-ddThh:mm:ss"
}
```

Apple also state:

    If the `TargetOSVersion` is an OS version that includes both a minor and patch version, 
    the system installs that specific version, for example, `16.1.1`. If the minor version 
    doesn't include a patch version, the system installs the latest available patch version. 
    For example, if the `TargetOSVersion` is `16.1` and a `.1` patch is available, the 
    system installs `16.1.1`.

    The system can only install a supplemental software update on a device that already has the 
    base OS version installed. For example, the system can only install a `16.1`(a) update on a 
    device that currently has `16.1` installed, but it can't install that update on a device 
    that has only `16.0` installed. To update to a supplemental version from an older base 
    version, use two configurations. Use the first configuration to update to the new base 
    version, and the second configuration to update the new base version to its supplemental 
    version.

[Source](https://github.com/apple/device-management/blob/release/declarative/declarations/configurations/softwareupdate.enforcement.specific.yaml#L82-L84)

I've interpreted this in a way that means if you push 26.3.1 to a macOS device that is _already on_ macOS 26.3.1, after the BSI has been released, it should identify there's a BSI version available, and enforce that.

**However**, I haven't had success with this in testing.<br>
This could be a bug, or it could be that I've misinterpreted the docs.

I've also not had success specifying the `TargetOSVersion` as `26.3.1(a)`, or `26.3.1 (a)`.<br>
Inspection of the unified logs shows an error with the target OS version format:

```
Failed to apply new configurations...VersionString:26.3.1(a)|BuildVersionString:
25D770870b|DetailsURL:(null)|companyName:(null)|NotificationsEnabled:YES), error: Error 
Domain=SUCoreError Code=9100 "[kSUCoreErrorDDMInvalidDeclarationFailure] Invalid 
declaration: target OS version (26.3.1(a)) has an invalid format"...
```
The same error appears using `26.3.1 (a)`

Huge shoutout to [Ade](https://github.com/unknownade) for the predicate that helped me find this:

```terminal
log show --predicate 'composedMessage contains "26.3.1(a)"' --info --debug --last 5m
```
<br>
<br>

These errors could be because there's no entry in the GDMF feed where the `ProductVersion` matches those formats.<br>
The BSI does exist, but the `ProductVersion` is listed as `26.3.1` (or `26.3.2` for the Neo):

```json
"macOS": [
  {
    "ProductVersion": "26.3.1",
    "Build": "25D771280a",
    "ProductVersionExtra": "(a)",
    "PrerequisiteBuild": "25D2128",
    "PostingDate": "2026-03-17",
    "ExpirationDate": "2026-06-18",
    "SupportedDevices": [
        "..."
    ]
  }
```
This means that you can't use the built-in Blueprints option to enforce BSI installation, but that doesn't mean that it's not possible...

## How do I do this using Blueprints?

Blueprints is Jamf's approach to managing Apple devices using DDM.<br>
If you're unfamiliar, you can read more about Blueprints on [Jamf's Blog](https://www.jamf.com/blog/introducing-blueprints/).

Given the errors above, we're going to need to push a custom declaration to the device for the declaration to apply successfully, and point the device to the BSI update details.

To do this, create a new Blueprint (or amend a suitable existing one!) and search for "_Custom Declarations_" in the Components Library.


![Image of Jamf Pro Blueprint editor with the Custom Declarations selected in the components library.](/assets/img/postImages/2026-03-20/01-Custom-Blueprint.png)

Create the Custom Declaration, using the relevant declaration type, and payload.
![Image of Jamf Pro custom declaration using the com.apple.configuration.softwareupdate.enforcement.specific declaration type, and a payload containing the BSI build version and OS version](/assets/img/postImages/2026-03-20/02-Blueprint-Details.png)

The image above, I've been pushing the `26.3.1(a)` BSI, for macOS devices that _aren't_ the Neo.

Declaration type:
```
com.apple.configuration.softwareupdate.enforcement.specific
```
Payload:
```json
{
  "TargetOSVersion": "26.3.1",
  "TargetBuildVersion": "25D771280a",
  "TargetLocalDateTime": "2025-09-20T12:30:00"
}
```

Once that's entered, update the details, and add them to the Blueprint
![Image showing the confirmation of the custom declaration configuration](/assets/img/postImages/2026-03-20/03-Details-Added.png)

Then you're ready to configure the rest of your Blueprint, add the scope, and deploy.
![Image showing the Blueprint is ready for deployment](/assets/img/postImages/2026-03-20/04-Blueprint-Ready.png)



## How can this be done _without_ Blueprints?

To achieve this without Blueprints, it's slightly trickier as Jamf Support need to enable the _Custom version entry_ options within the Software updates section of Jamf Pro.

Without that, Jamf will push the standard 26.3.1 OS version, and the plan will fail with error: `DowngradeNotSupported`

![Image of Jamf Software update plan pushing 26.3.1](/assets/img/postImages/2026-03-20/05-SU-Non-custom.png)
![Image of plan showing failure on computer record with failure reason of DowngradeNotSupported](/assets/img/postImages/2026-03-20/06-Non-Custom-Fail-Message.png)

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
>This failure reason is found on the Computer record, under the Operating System History details.
{: .prompt-info }

<!-- markdownlint-restore -->

Once you have the _Custom version entry_ option enabled on your Jamf Pro tenant, you can then populate a plan with the relevant details, and see the plan progress successfully;

![Image of Jamf Software update plan pushing 26.3.1 with a custom build version entry](/assets/img/postImages/2026-03-20/07-SU-Custom.png)
![Image of plan showing progressing successfully on computer record](/assets/img/postImages/2026-03-20/08-Custom-Success-Message.png)

## What do users see?

As this is essentially pushing a DDM SU plan, users will see a _Managed Update_ notification advising them an update has been scheduled:
![Close up image of a Unified Notification Centre notification advising of a managed update to 26.3.1 (a)](/assets/img/postImages/2026-03-20/09-BSI-Update-UNC.png)

In System Settings, BSIs ***do not*** show up under General > Software Update.<br>
They instead live under Privacy & Security > Background Security Improvements:
![Image of System Settings open to the Background Security Improvements extension](/assets/img/postImages/2026-03-20/11-BSI-System-Settings.png)

## Credits

This post was born out of a couple of threads on the [MacAdmins Slack](https://www.macadmins.org/).

All the testing in this post is my own, but it was definitely helped by a few people, and I wanted to thank them for that - even if they didn't realise they helped:
- [Colorenz](https://github.com/colorenz)
- [charliwest](https://github.com/charliwest)
- [Aiden](https://github.com/aidentopp)
- [Mark Buffington](https://github.com/markbuffington)
- [adamcodega](https://github.com/acodega)

Special mention has to go to [Ade](https://github.com/unknownade), who's technical curiosity matches my own and helped to have an in depth, provocative conversation that led to this post.<br>



## Final thoughts...

Ultimately, Blueprints is the future for declarative management from Jamf Pro and so the non-Blueprints DDM SU process may eventually be removed.<br>

This guide is designed to give an option to admins who may not be able to move to Blueprints yet, either due to the OIDC-Based Single Sign-On with Jamf Account requirement, or for other reasons.<br>
Me personally, I can't yet enable that as it breaks other things, but those are internal challenges that I'm working through.

It's great to be able to maintain security when BSIs come along, without _needing_ Blueprints to be available.

Anyway, that's it for a Birthday Blog-post, I do hope it was helpful...I'm off for some cake.

See, ya!
