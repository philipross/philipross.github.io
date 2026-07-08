---
title: "Look Ma, No MDM commands! Deploying VPP apps using DDM"
date: 2026-07-13 11:00:00 +0100
description: "DDM is the standard in Device Management, and you can move to install App Store apps on your devices via Declarative Device Management."
categories: [Mac Management, Deployments]
tags: [macOS, VPP, DDM, Blueprints]
---

## VPP via DDM? When did this come about?

Apple actually released the `AppManaged` Declaration with the release of iOS/iPadOS 17.2, and it came to macOS with the macOS 26 Tahoe release. You can read more about it in the [Apple Developer Docs](https://developer.apple.com/documentation/devicemanagement/appmanaged).

I hadn't really looked into this too much when it was first released, but since Apple announced during WWDC '26 that Declarative device management is *the standard* for Device Management with the AppleOS 27 releases, this triggered the curiosity and got my time-to-test juices flowing!

By way of disclaimer - all of the testing shown in this post has been done on macOS 26 Tahoe.

## How do I do it?

In my testing, there's certain conditions that need to be set before an App Store app will install through DDM.<br>
I'm not sure if these are a result of using a Custom declaration, or how Jamf have implemented VPP.

*Or* if this is a result of how the `com.apple.configuration.app` declaration type works on the macOS Platform.

However, those conditions are as follows:

**Firstly** the app *must* be available in your MDM already, so you must have gone through Apple Business, purchased licenses, and assigned them the appropriate Organisational Unit.
  - This is a fairly obvious point, but I'll mention it anyway. There's no change here to how you do it today via the MDM channel.

**Secondly**, the device(s) you're sending the Declaration to *must* be in scope of the App ***before*** pushing the declaration to the device.<br>
  - If you push the declaration to the device before the device is in scope of the App, it won't install. 
  - If you subsequently move the device into scope after the declaration has landed on the device, it still won't install.
    - In that situation, you'd need to remove the declaration from the device, and re-push it to achieve the correct order or operations for the app to install.

### Got it. So...how do I do it?

Now we've got the pre-reqs out of the way, let's get to it!

For my testing, I've added Slack, and Windows App to the VPP applications available in my Jamf Pro server:
![Jamf Pro Mac Apps section showing Slack for Desktop, and Windows app under the App Store header](/assets/img/postImages/2026-07-13/01-Jamf-Pro-VPP-List.png)

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Note that Windows app has "No scope defined". This will allow me to demonstrate the behaviour when a declaration is installed, but the device is not in scope of the App.
{: .prompt-info }

<!-- markdownlint-restore -->

These Apps are configured as *"Make Available in Self Service"* as the Distribution method, and on my testing device I can see Slack listed in Self Service.<br>
I've also opened Finder to the `/Applications` directory, and we can see Slack is not listed:
![Jamf SSP showing Slack as available with a button to install, and Finder showing the Applications directory, without Slack being present](/assets/img/postImages/2026-07-13/02-SSP-and-Finder.png)

On the device, there's only one declaration showing for Software Update settings, which is not relevant for this post.

At the time of writing, in Jamf Pro, there is no pre-built Blueprint component available for the `com.apple.configuration.app` declaration type.<br>
You can see the declaration types are listed in alphabetical order, and the first one is `com.apple.configuration.audio-accessory.settings`, so it should be before that:
![A blank Jamf Pro Blueprint showing the components filtered to Declarative configurations](/assets/img/postImages/2026-07-13/04-Blueprint-creation-Jamf-Pro.png)

For my Custom Blueprint, I've scoped it and deployed it to my test device:
![Slack deployment Blueprint showing deployed, and showing the contents of the custom declaration](/assets/img/postImages/2026-07-13/05-Blueprint-deployment-Jamf-Pro.png)

The contents of my custom blueprint is:

- Kind: **Configuration**
- Channel: **System**
- Type: **com.apple.configuration.app**
- Payload:
```json
{
  "AppStoreID": "803453959",
  "UpdateBehavior": {
    "AutomaticAppUpdates": "AlwaysOn"
  },
  "InstallBehavior": {
    "Install": "Required",
    "License": {
      "Assignment": "Device"
    }
  }
}
```

### What does it look like client side?

{%
  include embed/video.html
  src='/assets/img/postImages/2026-07-13/06-DDM-Installation-client-side.mp4'
  types='mov'
  title='Slack install via DDM - client side'
  autoplay=true
  loop=true
  muted=true
%}

In the recording above it shows that as soon as the Declaration lands on the device, there's an `Slack.appdownload` entry present in the `/Applications` directory.<br>
Once the installation is complete, this disappears and is replaced with the `Slack.app` bundle that we know and love.

This is a ***huge*** change from VPP installations over the MDM channel, as this displays progress of the installation directly to the user, without them using specific predicates within the unified logging system.

The App Declaration looks like this when expanded:
![Slack for Desktop declaration expanded client-side](/assets/img/postImages/2026-07-13/07-App-DDM-expanded.png)


### How do I build the Custom Declaration?

Apple's Developer Docs (for this specific Declaration)[https://developer.apple.com/documentation/devicemanagement/appmanaged] are a great place to start to learn how to structure the declaration payload required here.

There are some keys available that I've not included in this post, so this is not an exhaustive example of how this can be done.

To find the `AppStoreID` value, this is listed in the URL for the App, when viewed in the `App Store for Mac` in a browser
![App Store for Mac in a browser showing the Slack for Desktop app, with the ID in the URL highlighted](/assets/img/postImages/2026-07-13/08-AppStoreID-MAS-Web.png)
Or, you can also find this same URL on the App record within Jamf Pro:
![Jamf Pro Mac Apps section showing the Slack for Desktop general settings, with the ID in the URL highlighted](/assets/img/postImages/2026-07-13/09-AppStoreID-JamfPro-Mac-Apps.png)

## What about that Windows App demonstration?

Well remembered! (I didn't forget, but allow me some showmanship!)

I've not changed the scope from earlier - Windows App is still not scoped, and this is what happens if you push a Declaration for it in that situation:

{%
  include embed/video.html
  src='/assets/img/postImages/2026-07-13/10-Windows-App-DDM-client-side.mp4'
  types='mov'
  title='Windows App attempt to install via DDM - client side'
  autoplay=true
  loop=true
  muted=true
%}

How....anticlimactic.

Because the App wasn't in scope before the Declaration was deployed, the App didn't start to install.


## That's all there is to it!

Pretty simple stuff today, but hopefully useful to those who read it.

I'd also like to point out - whilst I've used Jamf for testing purposes and creation of screenshots, this should work with any MDM that allows Custom Declarations, or that has built a UI around the `com.apple.configuration.app` declaration type.

I don't have access to another MDM to test this, but I'd be happy to help you try this in your if you want to reach out.


