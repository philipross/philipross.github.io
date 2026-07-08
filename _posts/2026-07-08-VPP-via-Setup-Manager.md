---
title: "VPP via Setup Manager? Challenge accepted."
date: 2026-07-09 11:00:00 +0100
description: "Since the release of Jamf Setup Manager, it's not been possible to install App Store Applications through VPP whilst running Setup Manager....until now!"
categories: [Mac Management, Device Onboarding]
tags: [Jamf Setup Manager, macOS, Setup Assistant, VPP]
---

## Rewinding the clock

One of my first blog posts detailed the process I was using to workaround the limitations of the `jamf` binary that [Jamf Setup Manager](https://github.com/jamf/Setup-Manager/tree/main) works with, and how that prevented the ability to command VPP apps to install directly during Setup Manager.

If you haven't read that, or want to refresh your memory, you can find it here:<br>
 - [Onboarding like it's 2099: Setup Manager gets a sidekick](https://philipross.github.io/posts/Combining-Jamf's-Setup-Manager-and-macOS-Onboarding-for-a-rich-user-experience/)

 At a high level, this process was:<br>
 1. Jamf Setup Manager runs policies
 2. User signs in
 3. VPP applications are installed through [macOS Onboarding](https://learn.jamf.com/en-US/bundle/jamf-pro-documentation-current/page/macOS_Onboarding.html)


## If it ain't broke, don't fix it!

Sure - this approach is still absolutely a valid and working method of onboarding your devices, ready for users with all required applications installed.

BUT.

Since the release of [Jamf Setup Checklist](https://github.com/Jamf-Concepts/setup-checklist/tree/main), I've been trying to migrate as much of my macOS Onboarding configuration into Setup Checklist as I can, but I still wasn't able to do away with macOS Onboarding *entirely*, simply due to the VPP App installation challenges.

This is an obstacle that's been irritating me for a while, and whilst I was sat listening to the Jamf Nation Live London keynote, I had an epiphany!

- We can use Jamf Setup Manager to call policies...
- Policies can include a script...
- VPP Apps can be made as a Self Service install item...
- We can initate install programatically using the URL Scheme for Self Service...

See where this is headed?

Admittedly, I had actually thought of, and tested this process in March 2025 and shared this on the [Mac Admins Slack](https://macadmins.slack.com/archives/C078DDLKRDW/p1741008513379369?thread_ts=1741006453.784969&cid=C078DDLKRDW), but at the time the UX was ugly and I didn't spend much time trying to improve it.

However, being a Mac Admin requires constant learning, and learnings from my more recent post where I use [Jamf Setup Checklist to populate user details in Jamf Pro](https://philipross.github.io/posts/Jamf-Setup-Checklist-Populate-User-Details/) afforded me the ability to reattempt this.

## I'm intruiged....go on...

I set about testing this using Slack as the VPP app I wanted to install during Setup Manager.

To do this, I 
