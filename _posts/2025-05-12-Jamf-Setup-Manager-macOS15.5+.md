---
title: "Déjà Vu... How changes to macOS 15.5 impact the use of Jamf Setup Manager"
date: 2025-05-12 18:00:00
description: "Continuing on from my previous post, this post dives into the changes to Setup Assistant in macOS 15.5+ and how they impact the use of Jamf Setup Manager"
cateogories: [Jamf Setup Manager]
tags: [Jamf Setup Manager, macOS, Setup Assistant]
---


However, Apple have already released functionality within macOS to make this a moot issue and it's already live in Jamf Pro<br>(n.b: I don't have access to other Jamf Products to confirm if it exists there).<br>
[Link to Apple's Platform Deployment guide](https://support.apple.com/en-gb/guide/deployment/dep73069dd57/web#:~:text=Enforcing%20a%20minimum,put%20into%20production.)<br>
[Link to Jamf's PreStage documentation](https://learn.jamf.com/en-US/bundle/jamf-pro-documentation-current/page/Automated_Device_Enrollment_for_Computers.html#ariaid-title5)

To make this as straightforward as possible, you can leverage these settings to specify a Minimum OS version at enrolment, and either:
- Set it to 15.5 explicitly<br>
    ![Image showing ADE Minimum OS version set to 15.5 in Jamf Pro PreStage](/assets/img/postImages/ADE-Minimum-15.5.png)
- Set it to the Latest Major version available
    ![Image showing ADE Minimim OS version set to Latest Major Version in Jamf Pro PreStage](/assets/img/postImages/ADE-Minimum-LatestMajor.png)

By setting one of these options, *any* macOS client that's not on 15.5 should be forced to update it's OS during Setup Assistant, before enrolment into MDM can complete.
