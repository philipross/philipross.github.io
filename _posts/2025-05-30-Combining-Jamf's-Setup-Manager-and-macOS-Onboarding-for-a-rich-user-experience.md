---
title: "Combining Jamf's Setup Manager and macOS Onboarding tools for a rich user experience"
date: 2025-05-30 12:30:00 +0100
description: "Detailing a way to leverage a combination of Jamf's onboarding tools to elevate the user experience during device onboarding. This post explains how I combine Jamf Setup Manager, with Jamf Pro's macOS Onboarding to deliver a rich experience to end users."
categories: [Jamf Setup Manager]
tags: [Jamf Setup Manager, macOS, Setup Assistant, macOS Onboarding]
---

## What is the difference between Jamf Setup Manager, and Jamf macOS Onboarding?

[Jamf Setup Manager](https://github.com/jamf/Setup-Manager/tree/main) is a utility to help provide a set of configurations to a device *prior* to the user login.<br>
Similarly, but contrastingly, [Jamf macOS Onboarding](https://learn.jamf.com/en-US/bundle/jamf-pro-documentation-current/page/macOS_Onboarding.html) is a mechanism built into Jamf Self Service to deliver a set of configurations *after* the user has logged in.

Both tools have their pros and cons, their strengths and limitations, but by using a combination of the two we can counteract a limitation of one tool, with a strength of the other.

For the purposes of this post, I'll be specifically looking at the strengths and limitations below:

| Tool | Strength | Limitation |
| :----: | :-------- | :---------- |
| Jamf Setup Manager | - Runs prior to logon<br>- Can be used to deliver a standard 'stock build' with a one-touch build approach<br>- Many customisation options<br>- ***Excellent*** UI for user experience | - Runs in System Space (or a System User space)<br>- Unable to deliver VPP applications
| Jamf macOS Onboarding | - Able to deliver VPP applications<br>- Runs in user space as driven by Jamf Self Service<br>- Able to be retriggered in future[^1]  | - Runs after logon<br>- Limited customisation options<br>

[^1]: This is a planned topic for a future post...

