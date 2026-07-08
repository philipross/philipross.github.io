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

#### App setup

I set about testing this using Slack as the VPP app I wanted to install during Setup Manager, but you should be able to use any app that you wish to, provided the configuration is the same.

Firstly, we need to make sure that the VPP App is set to install through Self Service, and is scoped accordingly.

![Slack for Desktop app setting shown in Jamf Pro, highlighting Distribution Method is set to "Make Available in Self Service](/assets/img/postImages/2026-07-09/01-VPP-App-Config-Jamf-Pro.png)

We also need to grab the App ID which is both in the Jamf Pro URL, and also listed in the App URLs section under the Self Service settings. In the example image below, the App ID is `1`.

![Slack for Desktop Self Service settings shown in Jamf Pro, highlighting the app ID in the Site URL, and alos in the App URLs section](/assets/img/postImages/2026-07-09/02-VPP-App-SelfServiceConfig-Jamf-Pro.png)

#### Handling the installation

As I touched on earlier, to actually *do* the installation during Setup Manager, we need to initate this from a script contained in a policy.

The policy setup is simple:
- Scope it accordingly
- Give it a custom trigger to initiate it from Setup Manager
- Add the script as a policy payload
- Complete the required fields within the script parameters

The script is the heavy lifting bit here.<br>
I'll caveat that I'm ***definitely not*** a great scripting admin, but I've tried my best. If you can spot obvious improvements, please do let me know as I'd love to hear about them!

My idea was that the policy would initate the install and then watch for completion of the installation before marking the policy as complete. To prevent an issue during this from keeping Setup Manager running on a specific step indefinitely, I've included a timeout which will make the install as failed, and allow Setup Manager to progress.

The timeout threshold can be customised, but I'd recommend starting with a higher value and reducing it than the other way around.<br>
In my testing, I started around 10 minutes (600 seconds) and reduced it to 5 minutes (300 seconds).

```zsh
#!/bin/zsh

# This script is designed to allow for VPP apps to install during Jamf Setup Manager.
# This will be done by calling the url scheme to execute the install of the app, using the `open -j` argument to hide the SSP app GUI

#Variables that users can set
URL_SCHEME=$4
ID=$5
APP_PATH=$6
TIMEOUT_THRESHOLD=$7
SLEEP_VALUE=$8
SELF_SERVICE_NAME=$9


# Fixed variables
ELAPSED_TIME=0
URL=${URL_SCHEME}content?entity=app\&id=${ID}\&action=execute

# 1. Check for presence of the app.
# If it exists, exit the script with no action taken. Report success, as the app is present.
# If it doesn't exist, then call the Self Service URL to install silently

function quit_self_service(){
/bin/sleep 2
/usr/bin/osascript -e "tell application \"$1\" to quit"
}

if [[ -d "$APP_PATH" ]]; then
    echo "$APP_PATH exits."
    echo "Not calling to install. Exiting with success"
    exit 0
else
    echo "$APP_PATH does not exist."
    echo "Calling to install..."
    open -j "$URL"
fi

# Use an until loop to act as a watchpath
until [[ -d "$APP_PATH" ]];do
    if [[ ELAPSED_TIME -ge TIMEOUT_THRESHOLD ]]
        then
            echo "Timeout threshold reached."
            echo "Marking installation as failed."
            quit_self_service $SELF_SERVICE_NAME
            exit 1
        fi

    echo "$APP_PATH does not yet exist. Looping..."
    echo "Time elapsed: $ELAPSED_TIME"

    sleep $SLEEP_VALUE
    ((ELAPSED_TIME+=SLEEP_VALUE))
done

echo "$APP_PATH exists. Completing..."
quit_self_service $SELF_SERVICE_NAME

exit 0
```
I've also specified the Parameter labels as the following:

| Parameter   | Label             |
| :---------- | :---------------- | 
| Parameter 4 | URL_SCHEME        | 
| Parameter 5 | ID                |
| Parameter 6 | APP_PATH          |
| Parameter 7 | TIMEOUT_THRESHOLD |
| Parameter 8 | SLEEP_VALUE       |
| Parameter 9 | SELF_SERVICE_NAME |

![VPP Install script in Jamf Pro showing the parameter labels](/assets/img/postImages/2026-07-09/03-Script-Parameter-Labels.png)

For Slack, I then populated the following information:

| Parameter   | Label             | Value                   |
| :---------- | :---------------- | :---------------------- |
| Parameter 4 | URL_SCHEME        | jamfselfservice://      |
| Parameter 5 | ID                | 1                       |
| Parameter 6 | APP_PATH          | /Applications/Slack.app |
| Parameter 7 | TIMEOUT_THRESHOLD | 300                     |
| Parameter 8 | SLEEP_VALUE       | 5                       |
| Parameter 9 | SELF_SERVICE_NAME | Self Service+           |

![Policy script settings showing the parameter values](/assets/img/postImages/2026-07-09/04-Policy-Script-Parameter-Values.png)

### What does it look like in action?

Below you can see a recording of Jamf Setup Manager executing a very slimline set of actions. This is merely for demonstrating purposes, and I've sped the recording up so it's as quick to watch as possible.

At the end of the recording, I open `Finder`, and nagivate to the `/Applications` directory to show that `Slack.app` is present on the device.<br>
I also open the `/Applications/Slack.app` package contents to show the presence of the `_MASReceipt` directory, which is only present on installs of Apps from the Mac App Store.

{%
  include embed/video.html
  src='/assets/img/postImages/2026-07-09/05-JSM-Run-through-VPP.mp4'
  types='mov'
  title='Jamf Setup Manager installing Slack through VPP'
  autoplay=true
  loop=true
  muted=true
%}

Once that completed, we can see the policy logs report the output of the installation script, increasing in time and reporting a success when the `/Applications/Slack.app` path exists on the device.

![Slack install policy logs from Jamf Pro](/assets/img/postImages/2026-07-09/06-Post-JSM-Policy-Logs.png)

We can also see a Completed MDM command to `Install App - Slack for Desktop` in the Management history section of the computer record, further proving this has indeed been installed through VPP.

![Management command history showing a successful command to install Slack for Desktop](/assets/img/postImages/2026-07-09/07-Post-JSM-Management-Commands.png)

### Et voilà!

This testing wasn't without it's challenges, and whilst this does work in this example, I'd recommend ***thorough*** testing if you want to use this in your environment.

Since I worked this process out, I've also had some subsequent thoughts on modernising this further which in early testing is proving successful.<br>
That'll be the focus of a future blog post, coming soon™️!