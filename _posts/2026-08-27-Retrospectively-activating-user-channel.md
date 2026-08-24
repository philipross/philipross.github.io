---
title: "Back to the Future: Retrospectively activating the macOS User Channel post-enrolment"
date: 2026-08-27 08:00:00 +0100
description: "For many organisations, their current macOS provisioning process doesn't make the user MDM-enabled. Here's an example of how you can activate that for your already enrolled devices."
categories: [Mac Management]
tags: [Jamf, macOS, DDM, Blueprints, MDM Migration, Apple Business]
---

## What's an *MDM-Enabled* user?

When looking at applying configurations to Apple Platforms, Apple gives us details on the Configuration Availability for which Platforms a control is applicable to.

Taking the [Safari Extension declarative settings](https://developer.apple.com/documentation/devicemanagement/safariextensionsettings#Configuration-availability){:target="_blank"} as an example:

![The configuration availability of Safari Extension declarative controls taken from Apple's Developer docs](/assets/img/postImages/2026-08-27/1-Safari-Extension-Configuration-Availability.png)

We can see that this declaration is applicable to the system scope on iOS, and visionOS, but only the user scope on macOS and Shared iPad.<br>
As this control is applicable to macOS *only on the user scope*, it can only be applied if there is an MDM-Enabled user on the macOS device.

iOS doesn't really have the concept of "user accounts", so doesn't have a user scope.<br>
iPadOS does, but only if it's set up to be Shared iPad, which requires specific configuration to enable.

If there is no MDM-Enabled user on the Mac, admins cannot the user channel is not active, so no user configurations can be applied.

## How does a user become MDM-Enabled?

If you're provisioning your devices using Automated Device Enrolment (ADE) into a Device Mangement Service (DMS), your users may already be MDM-Enabled.<br>
If you're not sure, [Rich Trouton](https://github.com/rtrouton){:target="_blank"} has a great Derflounder [post](https://derflounder.wordpress.com/2025/10/18/identifying-mdm-managed-user-accounts-using-system-information-on-macos-tahoe/){:target="_blank"} on how to identify MDM-Enabled users on macOS Tahoe which you could use on a fresh enrolment in your Org to check.<br>

However, if you - like me - are skipping user creation in Setup Assistant, and handing JIT account creation over to a tool like Jamf Connect, your macOS users are most likely *not* MDM-Enabled.

And this presents a big headache.

## Ah crap.  My users aren't MDM-Enabled...what do I do?

At the time of writing, Apple haven't formally given admins a way to retrospectively enable the user channel on macOS devices, and make existing users MDM-Enabled.

It is possible to do it with the `profiles renew -type enrollment` command, however that comes with it's own constraints:

- If a device is already enrolled in a DMS, running this command will renew enrolment, but will require admin credentials to authenticate the re-enrolment in System Settings.
- If a device is ***not*** enrolled in a DMS, this command does not require admin credentials.
    - This did require admin credentials prior to macOS Sequoia. [Source](https://support.apple.com/en-md/121011#:~:text=profiles%20renew%20%2Dtype,enrolled%20in%20MDM.){:target="_blank"}

With those constraints taken into consideration, it sounds like we have 3 options:
1. Modify our enrolment process, then wipe and re-enrol all our devices.
2. Run `profiles renew -type enrollment` on all existing devices providing admin credentials to standard users, or elevating their permissions to do so.
3. Remove the MDM profile from existing devices, and instruct users to run the re-enrolment themselves.

None of these options sound great.<br>
They either require a lot of work for admins, or are a high-risk approach involving elevating users to admin,  or temporarily losing management of devices and hoping users can/will run the re-enrolment themselves.

sad-macadmin.png.

<br>

# However.

There *is* a 4th option, which:
- [x] Does not require wiping and re-enrolling
- [x] Does not require modifying users permissions on their devices
- [x] Does not leave devices in an unmanaged state for any period of time.

I have two words for you: MDM Migration.

When Apple released their AppleOS 26 releases, they also released a feature brand-new in the-product-formerly-known-as-Apple-Business-Manager (Apple Business) that allows admins to enforce devices migrate from one DMS, to another, without needing to wipe their devices.

If you aren't aware of this feature, here are the relevant pages from the [Deployment Guide](https://support.apple.com/en-gb/guide/deployment/dep4acb2aa44/web){:target="_blank"} on the topic.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->

>The astute amongst you may well have spotted that this blog is heavily leant towards how to do things in Jamf. There's good reason for this - it's the only DMS I have access to.<br>
>From herein, this post will be no exception, however I do hope that for admins reading this who use a different flavour of DMS, it's given you some inspiration to investigate how you could do this in the tooling you use.<br>
>I'd be really interested to hear if you're able to leverage this approach in your environments.
{: .prompt-warning }

<!-- markdownlint-restore -->

## Sidebar

Before we carry on, I wanted to explain my thinking.
<br>


I think there's two problems that need to be solved here.

1. Firstly, we need to make sure that any *new* enrolments are MDM-Enabled. 
    - That's not the focus of this post, but really it should be the first thing you do. You don't want to be adding water into the bucket, whilst we're trying to empty it.
2. Secondly, once you have that nailed, we need to retrofit this to our existing devices.
    - That's what this post is focussing on.

To support the retrospective enablment of the user channel via MDM-Migration, we'll need to create a new PreStage in Jamf Pro.

This new PreStage will also give an opportunity to address point 1 above, _and_ enable point 2.

After this has been done, it's possible that this new PreStage may also become your default moving forward, so ensuring devices enrolled using this new PreStage ensure your users become MDM-Enabled from the get-go is important.


## Sounds like a lot of work...

Honestly, it's not!

I have quite a simple Jamf environment, and I also don't do any scoping based on pre-stage assignment.

However I think this might be relatively straightforward for even the most complex environment.

To use the MDM Migration feature in Apple Business, we need to have a different DMS to assign the device to.
So, first step is to create that, and link the ADE token into Jamf Pro.<br>
 - I won't cover that here, it's pretty bread and butter, and if you're already using ADE you'll have done it already. Never the less, here's the [docco from Jamf](https://learn.jamf.com/r/en-US/jamf-pro-documentation-current/Automated_Device_Enrollment_Integration){:target="_blank"}

Once you've done that, I found the easiest step to take next was to clone your existing pre-stage. This keeps all of the existing configuration the same, but you will need to update the ADE Instance the pre-stage is linked to, and tick the "Automatically assign new devices" box.

![Cloned pre-stage with the relevant sections highlighted](/assets/img/postImages/2026-08-27/2-Cloned-Prestage.png)

Once we've got that done, we can modify the device assignment in Apple Business, set a deadline, and await the notification on the device to begin the process.
If you don't see the option to set a deadline, then you'll need to ensure your device(s) meet the [requirements.](https://support.apple.com/en-gb/guide/business/axm3a49a769d/web#axmc1e982710){:target="_blank"}

### What does it look like on the device?

Before we complete the migration, I took a screenshot to prove two things:
1. The current logged in user is ***not*** an admin on the device. 
    - We need this to work with standard users *and* admin users, and this proves it works for standard users.
2. There is no MDM-Enabled user on the device already. 
    - Running the command from Rich's post I linked earlier shows no results.

![Terminal showing the two commands proving logged in user is not admin, and no MDM-Enabled user on system](/assets/img/postImages/2026-08-27/3-PreMigration-Terminal.png)


When the MDM Migration prompt first appears for a user, they will see a notification in Notification Center:
![Notification Center prompt for Migration starting](/assets/img/postImages/2026-08-27/4-Migration-Notification.png){: width="872" height="458" .w-75}

