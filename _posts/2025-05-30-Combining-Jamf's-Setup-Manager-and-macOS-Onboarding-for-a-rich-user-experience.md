---
title: "Combining Jamf's Setup Manager and macOS Onboarding tools for a rich user experience"
date: 2025-05-30 10:10:00 +0100
description: "Combining Jamf’s Setup Manager and macOS Onboarding tools can enhance the user experience during device onboarding. This post explains how I integrate Jamf Setup Manager with Jamf Pro’s macOS Onboarding to provide a rich experience for end users."
categories: [Mac Management, Device Onboarding]
tags: [Jamf Setup Manager, macOS, Setup Assistant, macOS Onboarding]
---

## What is the difference between Jamf Setup Manager and Jamf macOS Onboarding?

[Jamf Setup Manager](https://github.com/jamf/Setup-Manager/tree/main) is a utility that sets device configuration *before* the user logs in. <br>
In contrast, [Jamf macOS Onboarding](https://learn.jamf.com/en-US/bundle/jamf-pro-documentation-current/page/macOS_Onboarding.html) is a mechanism within Jamf Pro that sets configurations *after* the user logs in, through the Self Service app.

Both tools have their advantages and disadvantages, but by combining them we can overcome the limitations of one tool with the strengths of the other.

## What can Jamf Setup Manager do...or not do?

Jamf Setup Manager offers a range of capabilities, as detailed in its extensive documentation on the [Setup Manager GitHub page](https://github.com/jamf/Setup-Manager/tree/main).<br>However, to understand its limitations, it’s crucial to understand how it operates in the background.

Setup Manager runs actions by using the `jamf` binary. Policies run by executing the `policy` command using the `-event` argument, requiring a custom trigger to be set on the Jamf policy.

However, Setup Manager relies on the `jamf` binary's capabilities.

The `jamf` binary lacks a mechanism to run Jamf App installers, or install VPP applications[^1], preventing Setup Manager from initiating these actions.

[^1]: This information is accurate at the time of writing


Setup Manager is designed to run ***before*** the user logs in to their device, either during Setup Assistant, or over the login window.<br>This means policies cannot run scripts used for setting items within the user space, such as a `dockutil` script configuring the dock, or `desktoppr` scripts for setting the wallpaper.

Whilst you can create a LaunchAgent to run scripts upon user login and install that LA via a policy run in Setup Manager, this won't address the issue of the VPP application installs....


## What about Jamf macOS Onboarding?

Jamf macOS Onboarding is driven by Jamf Self Service, and *must* run in an interactive user session.

Since we can add VPP applications to Self Service for users to selectively install, we can leverage this configuration to have macOS Onboarding install any VPP-delivered applications.

macOS Onboarding also has the ability to be re-triggered in the future, on already enrolled devices. There's more details covered on this process in my post:<br>
- [Re-triggering Jamf macOS Onboarding to provide an engaging interface to users when deploying software
](https://philipross.github.io/posts/Retriggering-macOS-Onboarding/)


## So, which is better?

I don’t intend to declare one tool as superior to the other. This post aims to explain how you can combine both tools to overcome their limitations.


## My experience

Before migrating my device onboarding flow to Jamf Setup Manager, all of my onboarding was delivered via Jamf macOS Onboarding.<br>
This was good experience, but using a one-touch technician build process meant we had to wait until the user arrived to sign in for the build to be finished.<br>
Alternatively, we could sign in with the local admin account to complete the build, but this presented other challenges...

Now, we've taken the leap and moved to Jamf Setup Manager to deliver the core build of our devices, however we've still left macOS Onboarding enabled and doing *some* tasks.<br>
I do still have all of my policies and actions which Setup Manager runs included in onboarding as a fail-safe for any policy failures, or network issues that cause Setup Manager to fail.

Here's Setup Manager in action:
![Jamf Setup Manager running a few actions](/assets/img/postImages/2025-05-30/build-SetupManager.png)


But the main items that macOS Onboarding delivers are:
- VPP Applications
- Scripts that set configurations within the user space, including dock configuration, wallpaper, post enrolment user input, and more.


Here's how macOS Onboarding looks in action:
![Jamf macOS Onboarding running 4 items](/assets/img/postImages/2025-05-30/build-macOSOnboarding-full.png)

In the image above, you can see that Finder is active, and there's a Dock process running, indicating that macOS Onboarding is running within an active user session.

You can also see that I've got two applications being installed within the macOS Onboarding process: Slack and Windows App.

Both of these are delivered via VPP to my devices, and I've included them in the macOS Onboarding configuration in Jamf Pro.<br>This way, when I have my dock policy run the `dockutil` script, I don't end up with two question marks in my dock, instead of the application icons.



## What does my macOS Onboarding configuration look like?

Here's a snip of my Jamf macOS Onboarding configuration that delivers what's seen above:
![Image of Jamf Pro macOS Onboarding configuration taken from the Web UI](/assets/img/postImages/2025-05-30/build-macOSOnboarding-configuration.png)

I mentioned earlier in the post that I still have all of the policies that Jamf Setup Manager delivers included in Jamf macOS Onboarding for a fail-safe, which is why the items included in the Jamf Setup Manager image are also seen here.<br>
This isn't necessary, but I find it useful to have a backup in case of an interruption during device build that causes an issue.


## Why should you do this?

Ultimately, you don't have to if you don't want to, or if it doesn't work for your setup!

If you're looking at streamlining your onboarding by implementing Jamf Setup Manager, but you need items to run in an active user session, or want to also deliver VPP applications, then this is one option that's available to you.

Other tools are, of course, available.
- DEPNotify
- Outset
- Setup Your Mac
- I'm sure there are others I've not listed here...

I've chosen macOS Onboarding as it's built-in to Jamf Pro and helps get users familiar with Self Service from the first time they use their device.<br>
As I was utilising it before the switch to Jamf Setup Manager, it made sense to continue using it rather than switch to a different tool.


## Links & References

1. My `Set Wallpaper` policy uses [desktoppr](https://github.com/scriptingosx/desktoppr), written by [Armin Briegel](https://github.com/scriptingosx)
2. My `Set Dock` policy uses [dockutil](https://github.com/kcrawford/dockutil), written by [Kyle Crawford](https://github.com/kcrawford)
