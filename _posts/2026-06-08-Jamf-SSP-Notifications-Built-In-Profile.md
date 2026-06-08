---
title: "Don't ignore me!: Self Service+ notifications are now natively managed on all devices."
date: 2026-06-08 11:00:00 +0100
description: "Notifications for Jamf's Self Service+ application were only managed for devices enrolled after Jamf Pro 11.18, or via a custom profile. Jamf Pro 11.28.x version fixes this across your fleet"
categories: [Mac Management]
tags: [macOS, Jamf, Self Service+]
---

## What's new?

Hidden within the Jamf Pro 11.28 [release notes](https://learn.jamf.com/r/en-US/jamf-pro-release-notes-current/Resolved_Issues) there is a small entry that will bring a tear of joy to an admin's eye:

```
[PI136028] Fixed: Jamf Pro does not install the Jamf Notifications profile to 
already‑managed computers after a Jamf Pro upgrade.
```

Initially I missed this nugget, and only looked into it as a result of a thread comment posted by [Fraser](https://github.com/fraserhess) on the [MacAdmins Slack](https://www.macadmins.org/).


### What does this mean?

When Jamf released Jamf Pro 11.18, they included a change that if you were already [automatically installing a notification profile](https://learn.jamf.com/r/en-US/jamf-pro-documentation-11.18.0/Automatically_Installing_a_Jamf_Notifications_Profile), this profile would then include management of the `Self Service+` notifications, as well as the already included `Self Service (classic)`.

The catch? This logic only applied to *new* enrolments hitting the Jamf Pro server after it was upgraded.<br>
Already managed macOS devices then needed a custom profile to manage the Self Service+ notifications, to avoid prompts to end users.

![Image of Jamf Pro documentation highlighting that a manual notification profile is needed for devices enrolled into Jamf Pro 11.17.1 or earlier](/assets/img/postImages/2026-06-08/1-Jamf-Notes-11.18.png)

I've verified this on a device that was enrolled before 11.17.1, and has *only* the built in notifications profile enabled.<br>
Taking a peek at the contents of the profile, this is what is contained:

![Image of terminal showing the contents of the profile in XML structure](/assets/img/postImages/2026-06-08/2-Profile-contents-terminal.png)

```xml
...
    <key>ProfileIdentifier</key>
    <string>com.jamf.notifications.settings</string>
    <key>ProfileInstallDate</key>
    <string>2024-04-24 14:46:49 +0000</string>
    <key>ProfileItems</key>
    <array>
        <dict>
            <key>PayloadContent</key>
            <dict>
                <key>NotificationSettings</key>
                <array>
                    <dict>
                        <key>AlertType</key>
                        <integer>2</integer>
                        <key>BadgesEnabled</key>
                        <true/>
                        <key>BundleIdentifier</key>
                        <string>com.jamfsoftware.selfservice.mac</string>
                        <key>NotificationsEnabled</key>
                        <true/>
                        <key>ShowInLockScreen</key>
                        <true/>
                        <key>ShowInNotificationCenter</key>
                        <true/>
                    </dict>
                    <dict>
                        <key>AlertType</key>
                        <integer>2</integer>
                        <key>BadgesEnabled</key>
                        <true/>
                        <key>BundleIdentifier</key>
                        <string>com.jamfsoftware.Management-Action</string>
                        <key>NotificationsEnabled</key>
                        <true/>
                        <key>ShowInLockScreen</key>
                        <true/>
                        <key>ShowInNotificationCenter</key>
                        <true/>
                    </dict>
                </array>
            </dict>
...
```
{: file='com.jamf.notifications.settings.mobileconfig'}

I also verified there was no entry for anything Self Service+ related with a query of the managed notification plist file using a `defaults` command:

```zsh
defaults read /Library/Managed\ Preferences/com.apple.notificationsettings.plist | grep -i jamf
            BundleIdentifier = "com.jamfsoftware.selfservice.mac";
            BundleIdentifier = "com.jamfsoftware.Management-Action";
```



### So how does it update?

After the Jamf Pro console has updated to 11.28.x and the Jamf Binary has updated on the device, I noticed there was a management command queued up to install the notifications profile.

![Computer record management history tab in Jamf Pro, searching for 'notifications'](/assets/img/postImages/2026-06-08/3-Queued-up%20computer-command.png)

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
>Note: This command only appeared for me after an Inventory Update was completed. Your experience may differ!
{: .prompt-info }

<!-- markdownlint-restore -->

### What's different on the device?

As this is now showing as installed on my device, we can query the profile contents again to see if it's changed.

![Terminal showing the contents of the updated profile in XML structure](/assets/img/postImages/2026-06-08/4-Updated-profile-contents-terminal.png)


```xml
...
    <key>ProfileIdentifier</key>
    <string>com.jamf.notifications.settings</string>
    <key>ProfileInstallDate</key>
    <string>2026-06-08 07:48:23 +0000</string>
    <key>ProfileItems</key>
    <array>
        <dict>
            <key>PayloadContent</key>
            <dict>
                <key>NotificationSettings</key>
                <array>
                    <dict>
                        <key>AlertType</key>
                        <integer>2</integer>
                        <key>BadgesEnabled</key>
                        <true/>
                        <key>BundleIdentifier</key>
                        <string>com.jamfsoftware.selfservice.mac</string>
                        <key>NotificationsEnabled</key>
                        <true/>
                        <key>ShowInLockScreen</key>
                        <true/>
                        <key>ShowInNotificationCenter</key>
                        <true/>
                    </dict>
                    <dict>
                        <key>AlertType</key>
                        <integer>2</integer>
                        <key>BadgesEnabled</key>
                        <true/>
                        <key>BundleIdentifier</key>
                        <string>com.jamf.selfserviceplus</string>
                        <key>NotificationsEnabled</key>
                        <true/>
                        <key>ShowInLockScreen</key>
                        <true/>
                        <key>ShowInNotificationCenter</key>
                        <true/>
                    </dict>
                    <dict>
                        <key>AlertType</key>
                        <integer>2</integer>
                        <key>BadgesEnabled</key>
                        <true/>
                        <key>BundleIdentifier</key>
                        <string>com.jamf.selfserviceplus.agent</string>
                        <key>NotificationsEnabled</key>
                        <true/>
                        <key>ShowInLockScreen</key>
                        <true/>
                        <key>ShowInNotificationCenter</key>
                        <true/>
                    </dict>
                    <dict>
                        <key>AlertType</key>
                        <integer>2</integer>
                        <key>BadgesEnabled</key>
                        <true/>
                        <key>BundleIdentifier</key>
                        <string>com.jamfsoftware.Management-Action</string>
                        <key>NotificationsEnabled</key>
                        <true/>
                        <key>ShowInLockScreen</key>
                        <true/>
                        <key>ShowInNotificationCenter</key>
                        <true/>
                    </dict>
                </array>
            </dict>
...
```
{: file='com.jamf.notifications.settings.mobileconfig'}

If you also wanted, you can check the contents of the profile in the System Settings UI

{%
  include embed/video.html
  src='/assets/img/postImages/2026-06-08/5-System-Settings-UI-showing-profile-contents.mp4'
  types='mov'
  title='Verifying Native Self Service+ Notification Management'
  autoplay=true
  loop=true
  muted=true
%}

### Et voilà!

In this instance, all of this should be taken care of behind the scenes for your fleet, and if - like me - you missed the entry in the release notes you might not even know this was happening *at all*.

Now this is all built in natively, this means you should be able to remove your custom notification profile(s) for Self Service+, to help keep your environments organised and lean.
I know it's only one configuration profile...but you have to start somewhere!