---
title: "Re-triggering Jamf macOS Onboarding to provide an engaging interface to users when deploying software"
date: 2025-06-17 09:00:00 +0100
description: "Jamf’s macOS Onboarding tool can enhance user engagement by providing an informative interface during software deployment. It keeps users informed about progress and device changes. I’ll demonstrate how to achieve this using simple scripting from Jamf."
categories: [Mac Management, Device Onboarding]
tags: [Jamf Setup Manager, macOS, Setup Assistant, macOS Onboarding]
---

In a previous post, I explained how I use Jamf’s Setup Manager and macOS Onboarding tools to create a rich user experience when configuring a Mac for a user. If you haven’t read it, visit the [post](https://philipross.github.io/posts/Combining-Jamf’s-Setup-Manager-and-macOS-Onboarding-for-a-rich-user-experience/) to learn more.

I also mentioned in that post that I would share details on how to deploy new core software to already enrolled devices by re-triggering macOS Onboarding.


## Why would I want to re-trigger macOS Onboarding?

Let's face it. Enterprise, education, and other organisations managing macOS devices often face a _dynamic_ environment. Vendors may change frequently as business needs evolve, and core applications may need updates or modifications. Re-triggering macOS Onboarding helps ensure that users receive timely updates and changes.

There are plenty of examples I could give here: changing EDR agent; updating or migrating VPN client; deploying a new Video-Conferencing desktop application. For the purposes of this post, I'll keep it simple with a deployment of VSCode.

Jamf Setup Manager and macOS Onboarding are great tool, but they typically run once during device setup and then stop.<br>
While we can include the VSCode application in the Setup Manager configuration, this won't provide users on existing devices with an interactive experience when the software is installed on their Mac.<br>
Some users may not even be aware that the software has been installed, which could drive tickets to your help desk requesting installation....

Each organisation communicates with users in its own way and uses its own tone of voice. However, we know that even with the best intentions, users may miss communications. By re-triggering macOS Onboarding on the device they are using, we aim to reduce the number of missed communications as much as possible.

## Makes sense...so how do I do it?

There's a couple of steps required but it's all very simple and straightforward.

#### 1. Create the policy for the new Application in Jamf

The first component needed is the policy to install the App you're looking to deploy (I did say it was straighforward!)

The image below shows the Visual Studio Code policy enabled for deployment via Self Service. I want to deploy it to everyone, so I've scoped it accordingly.<br>
I'm also using the custom trigger to ensure the app is deployed immediately on any new device enrolments using Jamf Setup Manager.

![Image showing the Visual Studio Code policy setup in Jamf, scoped to 'All Managed Clients', enabled in Self Service, and setup with a custom trigger of 'jsmVSC'](/assets/img/postImages/2025-06-17/0-Policy-creation.png)

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> This process could equally be used for configuration profiles, or Application delivered from the Mac App Store, via VPP. Any component that macOS Onboarding supports can be used within this process.
{: .prompt-tip }

<!-- markdownlint-restore -->


#### 2. Modify the macOS Onboarding 

To include the new policy within the macOS Onboarding list of actions, it needs to be added to the relevant settings within Jamf Pro.

This image shows the _Visual Studio Code_ policy has been added into the macOS Onboarding list of actions. As a result, it will now be included on any devices that haven't had the policy run already, when macOS Onboarding is re-triggered.

![Image of Jamf's macOS Onboarding configuration showing Visual Studio Code included in the list of actions](/assets/img/postImages/2025-06-17/01-Onboarding-configuration-modification.png)

We can see that Visual Studio code didn't run when macOS Onboarding ran intially because it wasn't included in the list of actions.<br>

<img src="/assets/img/postImages/2025-06-17/02-onboarding-previous-run.png" alt="Image showing macOS Onboarding running on a device and does not include Visual Studio Code in list of actions" width="500"/><br>

#### 3. Create components to re-trigger macOS Onboarding

In Jamf's macOS Onboarding documentation there's a section for [_excluding computers from macOS Onboarding_](https://learn.jamf.com/en-US/bundle/jamf-pro-documentation-current/page/macOS_Onboarding.html#ariaid-title2).

This demonstrates that we can exclude already enrolled devices from executing macOS Onboarding by setting a specific key in a specific plist file.

File: _~/Library/Preferences/com.jamfsoftware.selfservice.mac.plist_<br>
Key: _com.jamfsoftware.selfservices.onboardingcomplete_<br>
Value: _TRUE_

If this was set, or if a device has already run macOS Onboarding, checking the key value would return the following:

```shell
username@computername ~ % defaults read ~/Library/Preferences/com.jamfsoftware.selfservice.mac | grep onboarding

    "com.jamfsoftware.selfservice.onboardingcomplete" = 1;
```

Since this is a boolean value, it stands to reason that setting it to _0_ would mean that Self Service wouldn't recognise macOS Onboarding as complete and it would attempt to run it again.

Here's how I script this from within Jamf Pro:

```shell
#!/bin/bash
# Check Dock or Finder are active
  while ! pgrep -q Dock || ! pgrep -q Finder; do
    echo "Dock or Finder is not active yet..."
    sleep 2
  done

# Set the path and key value for the preferences file
PLIST_PATH="/Users/$3/Library/Preferences/com.jamfsoftware.selfservice.mac"
ONBOARDING_KEY="com.jamfsoftware.selfservice.onboardingcomplete"

# Check if Self Service is running
SS_RUNNING=$(pgrep -fl "Self Service" | grep -v grep)

if [ "$SS_RUNNING" ]; then
  # Display a message with a timer to close Self Service
  dialog --ontop --title none \
    --message "Updates are required which will restart Self Service.\nThis action will begin when the timer ends, or you can press OK to continue now." \
    --icon /usr/local/digitaldesk/branding/digital-desk-notext-systemsettings.png \
    --timer 20 \
    --moveable \
    --button1text OK \
    --mini

  killall "Self Service"
else
  # Display a message with a timer to launch Self Service
  dialog --ontop --title none \
    --message "Updates are required which will launch Self Service.\nThis action begins when the timer ends, or you can press OK to continue now." \
    --icon /usr/local/digitaldesk/branding/digital-desk-notext-systemsettings.png \
    --timer 20 \
    --moveable \
    --button1text OK \
    --mini
fi

# Check the value of the onboarding key
KEY_VALUE=$(defaults read $PLIST_PATH $ONBOARDING_KEY)

if [ "$KEY_VALUE" == "0" ]; then
  # If the key value is already 0, do nothing
  echo "Key value is already 0, not setting to 0"
else
  # Set the onboarding key value to 0 (complete)
  sudo -u $3 defaults write $PLIST_PATH $ONBOARDING_KEY -bool NO
  echo "Onboarding key value has been set to 0"
fi

# Wait for 1 second before opening Self Service
sleep 1

echo "Opening Self Service"
open -a "Self Service"

exit 0
```

###### ***(This script could definitely be cleaned up, I know!)***

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> [swiftDialog](https://github.com/swiftDialog/swiftDialog) is a prerequisite of the script above
{: .prompt-info }

<!-- markdownlint-restore -->


Execute this script in a policy and it will re-trigger macOS Onboarding on the devices in scope.

## What does this look like to the end-user?

Reading the script above, you can see there's some processing in there to identify if Self Service is currently running, and a slight change to the message that appears based on the output.

If Self Service is running, the user sees this:<br>
![Image of Self Service running with a swiftDialog notification over the top, warning the user that Self Service is about to be restarted because of required updates](/assets/img/postImages/2025-06-17/1-Restart-notification.png)

Once the timer has finished, or the user has acknowledged the prompt, Self Service quits and relaunches.

If Self Service _isn't_ running, the user sees this:<br>
![Image of a swiftDialog notification warning the user that Self Service is about to be opened because of required updates](/assets/img/postImages/2025-06-17/2-Open-notification.png)

Once the time has finished, or the user has acknowledged the prompt, Self Service launches.

As the `com.jamfsoftware.selfservice.onboardingcomplete` key is set to `FALSE` before Self Service relaunches, macOS Onboarding is retriggered, and will run any policies that have been added to the macOS Onboarding configuration, and have not yet run on the device:

![Image of Self Service running macOS Onboarding running the VSCode policy](/assets/img/postImages/2025-06-17/3-Onboarding-in-progress.png)

## What else is required in the back-end?

All of the changes in the back-end are relatively routine for a software deployment activity.

If you were deploying software _without_ this method, you would already need to:
1. Create a policy to install the new software
2. Complete modifications to your onboarding process to include the new software (e.g. Jamf Setup Manager configuration profile)

To achieve what's shown in this post, you would also need to:
1. Set up a script and policy to retrigger macOS Onboarding, and to notify your users.
2. Modify your macOS Onboarding configuration to include the new software.

## Why would I do extra work to do this instead?

The script and policy to reset macOS Onboarding is reusable.

I actually have mine set to run once a month on all clients, regardless of whether there's something new deployed or not.<br>
This helps my users to get used to the update notifications and also acts as a catch-all safety net for any policies that may have had an issue in deploying cleanly first time around.

Having this run on a regular cadence and scoping new software to groups of devices allows for a phased approach, with increasing pool sizes.<br>
I recommend the [deployment group](https://derflounder.wordpress.com/2021/09/10/setting-up-software-deployment-groups-using-a-jamf-pro-extension-attribute/) approach written by [Rich Trouton](https://github.com/rtrouton), but there are countless permutations with scoping and policy options.


Because Jamf macOS Onboarding is driven by Self Service, any users who want to run the new software from their Self Service catalog, before the forced re-trigger of macOS Onboarding can simply run the policy when it suits them.<br>
If you want complete control over a deployment, you can implement more granular scoping of the install policies. Coupling this with the scoping of your macOS Onboarding re-triggering work will allow for a more structured rollout.

