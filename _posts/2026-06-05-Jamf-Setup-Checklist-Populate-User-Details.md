---
title: "User Details? Check!: Leveraging Jamf Setup Checklist to populate user details in Jamf Pro"
date: 2026-06-05 14:00:00 -0100
description: "Jamf Setup Checklist is a great tool for getting your users ready to go from the moment they're logged in, however sometimes organisations need to populate Jamf Pro inventory fields to ensure device configuration is complete. This post covers an example of how to do just that."
categories: [Mac Management]
tags: [macOS, Jamf, Jamf Setup Checklist]
---

## What is Jamf Setup Checklist?

If you're using Jamf and haven't heard of it, [Jamf Setup Checklist](https://github.com/Jamf-Concepts/setup-checklist/tree/main) is a new tool from Jamf, currently in Public Beta.<br>
Definitely go check it out and have a play if you haven't done so already.


### What does it do?

The docs for Setup Checklist are pretty comprehensive regarding its capabilities and what it is designed to do, but what's important for the detail of this post is that Setup Checklist activates upon user login, runs within the user session and therefore runs in _user context._

Initially I forgot this and tried to deliver this outcome via a .sh file delivered to the device via package, but the script would have needed `sudo` so that approach failed.

This post will cover how to populate the *User and Location* details within a Jamf Pro Computer record.<br>
The fields available here can be interacted with through the `jamf` binary using a `recon` command.

### Jamf binary commands

If you run `jamf help recon` on a system that has the `jamf` binary installed, you'll see a number of options available that can be used to populate specific inventory information.

For the purposes of this post, I'll be using the `-department` option, but the premise is the same for any option available.

For the experienced amongst you, you'll know that the majority of commands executed using the `jamf` binary require root authorisation, so either need to be called from a Jamf Policy, or run with `sudo`.<br>
This presents a challenge as we cannot run an elevated command directly within Setup Checklist, and so have to get a bit creative with how to accomplish this.

Luckily...Jamf allows us to programmatically action a specific policy without using the binary as long as it's available to run within Self Service.<br>
<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
>References to "Self Service" within this post are agnostic of which version you're using in your environment. Both Self Service+ and Self Service (classic) will work provided the relevant URL scheme is used.<br>
Screenshots have been taken from Self Service+ only.
{: .prompt-info }

<!-- markdownlint-restore -->

### Jamf Pro Components

There's a few things required to make this work. Jamf Setup Checklist is a prerequisite of this process, but this post isn't written to go through how to use Jamf Setup Checklist, so I've assumed you've already done that.<br>
I'm also using [swiftDialog](https://github.com/swiftDialog/swiftDialog) to create the UI, so that's a requirement too. 

What do I need?<br>
1. A script to create the dialog window
2. A policy to run that script
    - This policy _must_ be available to run in Self Service
3. Relevant information populated in Jamf Pro
    - This example will focus on the use of the `Department` field in Jamf Pro. Therefore, there must be Departments that match the details already populated within Jamf Pro. More information can be found on Jamf's [Buildings and Departments](https://learn.jamf.com/r/en-US/jamf-pro-documentation-current/Buildings_and_Departments) page.

#### The script

For fans of the Irish pop group, prepare to be disappointed.<br>Whilst your man can't be moved, it's possible to re-run this script to update information in Jamf Pro if users do move around your organisation should you want to do this.

I've created a script in Jamf Pro called `Set Department`
![Image of Jamf Pro Script editor with a script setting up a swiftDialog prompt to capture user input](/assets/img/postImages/2026-06-05/1-Script.png)<br>
This script is 'quick and dirty' to show the possibilities here. swiftDialog has ***plenty*** of customisation options for you to play around with to suit your organisation's needs.<br>
Here are full contents to give you an idea of the structure:

```zsh
#!/bin/zsh

#Static details
dialogPath='/usr/local/bin/dialog'
dialogTitle="Department Selection"
dialogMessage="Please complete the fields below to complete the department assignment"

# Set Dialog Options
dialogOptions=(
    --button1text "OK"
    --width 700
    --height 300
    --titlefont "size=28"
    --messagefont "size=14"
    --selecttitle "Select a department"
    --selectvalues "Department A, Department B, Department C"
    --position centre
)

# Set Dialog content
dialogContent=(
    --title "$dialogTitle"
    --message "$dialogMessage"
)

# Call the dialog, and capture the output in a variable
dialogOutput=$("$dialogPath" "${dialogOptions[@]}" "${dialogContent[@]}")

# Quit Self Service+ 
osascript -e 'tell app "Self Service+" to quit'

# Revert Jamf Setup Checklist window position
setupchecklist step script-user-details windowPosition center

# Parse the output to capture the department
department=$(echo $dialogOutput | grep "SelectedOption" | awk -F " : " '{gsub(/"/,"",$NF); print $NF}' )

# Update the department field in Jamf Pro with a recon command. Also echo it so that it's captured in the Jamf Pro Policy Logs
jamf recon -department "$department"
echo $department

# Update the Jamf Setup Checklist step to completed so it may continue.
setupchecklist status script-user-details canContinue
```
{: file='postinstall_2'}

I wrote the structure of this script referencing a [blog post](https://bigmacadmin.wordpress.com/2023/01/03/avoiding-eval-with-swiftdialog/) from the mighty [BigMac Admin](https://github.com/bigmacadmin) explaining how to avoid using eval when using swiftDialog.

#### The policy

Fairly straightforward - this policy is set up to initiate execution of the script above. This script is set to be available in Self Service, so I can initiate it using the [Self Service URL scheme](https://learn.jamf.com/r/en-US/jamf-pro-documentation-current/Jamf_Self_Service_for_macOS_URL_Schemes).<br>

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
>This policy is *not* configured to update inventory via the Maintenance option. This is because the script it calls already runs a `recon` action, so omitting that option avoids repeated inventory updates.
{: .prompt-tip }

<!-- markdownlint-restore -->

![Image of Jamf Pro Policies showing the "Set Department" policy. This policy is set to run the "Set Department" script, created earlier](/assets/img/postImages/2026-06-05/3-Policy.png)

Navigating to the Self Service tab within the policy, you can grab the Installation URL that is specific to this policy.
![Image of the "Set Department" policy on the "Self Service" tab, highlighting the specific Installation URL that's populated for this policy](/assets/img/postImages/2026-06-05/4-Policy-URL-Scheme.png)

#### Departments created in Jamf Pro

From the script content, you'll see that I'm only giving a user the choice to choose from three Departments:
* Department A
* Department B
* Department C

This is purely for demonstrative purposes, but to set these on the computer record as we need to, these departments must exist within Jamf Pro already.<br>
This image shows that I've already created them within the Departments settings in Jamf Pro.
![Image showing the Jamf Pro Departments settings with 3 departments created; Department A; Department B; Department C](/assets/img/postImages/2026-06-05/5-Jamf-Pro-Departments.png)

### Got all that done, what's next?

Now we've got all of the components ready, we can create the new configuration for Jamf Setup Checklist to call this policy.

This action leverages the [script step](https://github.com/Jamf-Concepts/setup-checklist/blob/main/Profile/ScriptStep.md) in Jamf Setup Checklist to call this policy via the Self Service+ URL Scheme.

This is a powerful step that's capable of doing a multitude of things, but remember that Jamf Setup Checklist is running in the *user space*, so any script actions it calls also execute in the *user context*.

Initially I forgot this and tried to deliver a .sh file via pkg that would handle the dialog creation, and subsequent publish to Jamf Pro, but the `jamf` binary needed `sudo` so that approach failed.

This is the content I've put into my Jamf Setup Checklist configuration profile to execute this action.

```xml
<dict>
    <key>kind</key>
    <string>script</string>
    <key>icon</key>
    <string>symbol:person.bubble</string>
    <key>identifier</key>
    <string>script-user-details</string>
    <key>image</key>
    <string>symbol:person.fill.questionmark</string>
    <key>message</key>
    <string>Please complete the following information</string>
    <key>title</key>
    <string>User details</string>
    <key>prepareScript</key>
    <string>
    if [ ! -e /Applications/Self\ Service+.app ]; then
        setupchecklist status script-user-details error
    fi
    </string>
    <key>buttonScript</key>
    <string>open 'jamfselfservice://content?entity=policy&amp;id=13&amp;action=execute'</string>
    <key>windowPosition</key>
    <string>right</string>
</dict>
```
{: file='com.jamf.setupchecklist'}

Before I run this, this is what my computer record looks like in Jamf Pro.
![Image of computer record in Jamf Pro looking at the User and Location section, with no department set](/assets/img/postImages/2026-06-05/2-No-Department.png)


Here's a clip of what this experience looks like to the user

{%
  include embed/video.html
  src='/assets/img/postImages/2026-06-05/6-Screen-Recording.mp4'
  types='mov'
  title='Setting Department Information from Jamf Setup Checklist'
  autoplay=true
  loop=true
  muted=true
%}

Now that this has been run, we can refresh the computer record and see that the department has successfully populated.
![Image of computer record in Jamf Pro looking at the User and Location section, now showing a department has been set](/assets/img/postImages/2026-06-05/7-Department-Set.png)

### Et voilà!

In my opinion, this process isn't the prettiest, but it works around limitations of user context and binaries requiring sudo as best we can.<br>
As tools evolve, we may have a different option available to us in the future where this process can be revisited to refine the UI, and when it comes to macOS there's often more than one way to achieve the same outcome.

There's an important gotcha with how I delivered this.<br>
My script is leveraging the `setupchecklist` CLI to update the Jamf Setup Checklist UI that the step has been completed, and to enable the 'continue' button. If the script fails for some reason, this continue button won't enable and the user could get stuck at this step without clear instruction on how to proceed.


That's all I've got for today, so until the next time!