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
| Jamf macOS Onboarding | - Able to deliver VPP applications<br>- Runs in user space<br>- Driven by Jamf Self Service<br>- Able to be retriggered in future[^1]  | - Runs after logon<br>- Limited customisation options<br>- Requires the device to be logged into to install core applications, so can skew reporting

[^1]: This is a planned topic for a future post...

## So, which is better?

I'm not aiming to declare that one tool is better than the other here, this post is an explanation of how you can combine both tools to counteract each other's limitations.


## What do I do?

Before I migrated my device onboarding flow to Jamf Setup Manager, all of my onboarding was delivered via Jamf macOS Onboarding.<br>
This was good, but using a one-touch technician build process this meant we had to wait until the user arrived to sign in for the build to be finished.<br>
***OR***, we could sign in with the local admin account to complete the build, but this presented other challenges...

Now we've taken the leap and moved to Jamf Setup Manager to deliver the core build of our devices. But we've still left macOS Onboarding enabled, and doing *some* tasks.<br>
I do still have all of my policies and actions which Setup Manager runs included in onboarding, as a fail-safe for any policy failures, or network issues which means Setup Manager fails to launch successfully.

Here's Setup Manager in action:
![Jamf Setup Manager running a few actions](/assets/img/postImages/2025-05-30/build-SetupManager.png)


But the main items that macOS Onboarding delivers are:
- VPP Applications
- Scripts that set configurations within the user space
    - Dock configuration
    - Wallpaper
    - Post enrolment user input
    - etc...

Here's how macOS Onboarding looks like in flow:
![Jamf macOS Onboarding running 4 items](/assets/img/postImages/2025-05-30/build-macOSOnboarding-full.png)

As you can see in the image, Finder is active, and there's a Dock running, so this is running within the session of the logged in user.<br>
My first policy, and the final policy, are both running scripts which set user contextual items:
1. Wallpaper - using [desktoppr](https://github.com/scriptingosx/desktoppr)
2. Dock items - using [dockutil](https://github.com/kcrawford/dockutil)

In the instance above, both *Slack* and *Windows App* are delivered via VPP integration, rather than a Jamf Policy.


## What does my macOS Onboarding configuration look like?

Here's an snip of my Jamf macOS Onboarding configuration that delivers what's seen above:
![Image of Jamf Pro macOS Onboarding configuration taken from the Web UI](/assets/img/postImages/2025-05-30/build-macOSOnboarding-configuration.png)

I mentioned earlier in the post that I still have all of the policies that Jamf Setup Manager delivers included in Jamf macOS Onboarding for a fail-safe, which is why the items included in the Jamf Setup Manager image are also seen here.<br>
This isn't necessary, but I find it useful to have a back-up incase of network failure that causes an issue during device build.


## Why should you do this?

Ultimately, you don't have to if you don't want to, or if it doesn't work for your set up!

If you're looking at streamlining your onboarding by implemention Jamf Setup Manager, but you need items to run in an active user session, or want to also deliver VPP applications, then this is one option that's available to you.

Other tools are, of course, available.
- DEPNotify
- Outset
- Setup Your Mac
- I'm sure there are others I've not listed here...

I've chosen macOS Onboarding as it's built-in to Jamf Pro, and helps get users familiar with Self Service from the first time they user their device.<br>
As I was utilising it before the switch to Jamf Setup Manager, it made sense to continue using it rather than switch to a different tool.