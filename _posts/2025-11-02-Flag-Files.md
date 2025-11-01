---
title: "Capture the flag (files): Turning deployment chaos into organised victory"
date: 2025-11-02 09:00:00 +0100
description: "Creating structured workflow deployments is key to running deployments that need a specific order of operations. With a simple script, and Extension Attribute, you can utilise existing capability within Jamf Pro to order your deployment, and enhance reporting across your macOS fleet."
categories: [Mac Management, Deployments]
tags: [macOS, Flag Files, Jamf, Workflows]
---

## Long time, no post...

It's been pretty quiet on here for the last few months. Entering beta season, working on a complex deployment, and also taking some time off to get married has meant I've been a bit occupied and not had time to write.

However, that complex deployment has helped me to learn something new, and I thought it'd be a great opportunity to write something again!


## What have I learned that's new?

In two words: ***Flag files***.

Occasionally I'll come across an application deployment which requires me to think more carefully about how to craft the order of operations to achieve success with the deployment whilst limiting impact to the end user. 

Device Management Systems (DMS) often combine the use of the MDM/DDM channel provided by Apple with a separate, sometimes proprietary mechanism to deliver components outside of these channels. With Jamf Pro, configuration profiles or blueprints are delivered over the MDM/DDM channels, whereas policies and (some) inventory information is orchestrated by the `Jamf` binary.

If you come across a deployment that needs to make use of both of these mechanisms, it can be challenging to combine them with a structured order. 

**This is where I've been experimenting with flag files.**

## What is a flag file?

A flag file is very much *'does what it says on the tin'*. <br>It's a file that sits on a device for no purpose other than to cause something to happen, or to _not_ cause something to happen!

Applications and Operating Systems will often use them as markers of a completed action, to prevent the process that completes that action from re-running.

[@scriptingosx](https://github.com/scriptingosx)'s Setup Manager makes use of a flag file, as explained in the [documentation.](https://github.com/jamf/Setup-Manager/blob/main/Docs/Extras.md#flag-file)<br>
In the case for Setup Manager, the flag file marks the completion of Setup Manager's actions, and it's presence prevents the application from relaunching.

## How will it help me order my deployments?

Once a flag file is created, you can make use of an extension attribute to report the status of it across your macOS fleet.

As Jamf updates extension attribute values with 
