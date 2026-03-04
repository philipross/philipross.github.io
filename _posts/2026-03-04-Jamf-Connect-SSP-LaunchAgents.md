---
title: "The Right Agent, at the right time: Understanding LaunchAgent conflicts between Jamf Connect, and Jamf Self Service+ "
date: 2026-03-04 19:30:00 -0100
description: "Jamf Connect is now bundled in with Self Service+, but ingrained ways of updating the Jamf Connect versions may now be causing issues in Enterprise environments. This post is designed to highlight the changes to the LaunchAgent that runs the Account Management features of Self Service+, formerly included in Jamf Connect."
categories: [Mac Management]
tags: [macOS, Jamf, Jamf Connect, Self Service+ ]
---

### What is Self Service+?

If you're reading this blog post, it's likely you already know what Self Service+ is.<br>
If you've been under a rock for 18 months...Jamf announced Self Service+ at JNUC 2024 and followed it up with this [blog post](https://www.jamf.com/blog/meet-self-service-plus/).

In the time that's passed since then, admins have been able to test the installation and functionality, and plan their migration for Self Service+ in their environments.

Self Service Classic is due to be deprecated at the end of this month (31st March 2026), and as this deadline fast approaches, it makes sense that there's increased activity on the MacAdmins Slack with questions and admins offering advice from their experiences.

One of the core features of Self Service+ (I'm just going to type SSP from here...) is that it provides a unified interface for Jamf's applications, which previously end users would need to locate individually. These applications, at the time of writing, include:
- Jamf Connect
- Jamf Trust
- Jamf Protect
- Self Service functionality

There are probably many blog posts already in existence about SSP, its features, its limitations, and documented migration paths.

This post is designed to focus on the SSP Account Management features - which previously used to be the `/Applications/Jamf Connect.app/` component of the Jamf Connect product - specifically the LaunchAgent(s) that exist with both versions of the product.

#### Firstly, terminology:

I'm probably going to use a lot of acronyms in this post, so I'll detail them below for reference.

| Acronym | Meaning          |
| :------ | :--------------- |
| SSP     | Self Service+     |
| SSC     | Self Service Classic                           |
| JC      | Jamf Connect (the non-login window component)  |
| JCL     | Jamf Connect Login (the login window component)|
| JC 2.X  | Versions of Jamf Connect 2.X that bundle the login window and menu bar component together.<br>This has a ceiling of v2.45.1|
| JC 3.X  | Versions of Jamf Connect 3.X that only include the Login Window component<br> (because the menu bar components are within SSP) |

## What used to happen?

When installing JC 2.X, the DMG contained the installer pkg, but also a LaunchAgent pkg.<br>
This was required to ensure that the menu bar icon for Jamf Connect was shown and re-launched if the user attempted to quit the app.

Some admins often updated this when updating their versions of JC 2.X and re-deployed it, but the content of the LA didn't change and re-deploying it was often not required.


## What happens now?

Different versions of SSP have different behaviour with respect to the `/Applications/Jamf Connect.app/` directory.

Versions of SSP up to and including 2.14 used to replace the `/Applications/Jamf Connect.app/` directory with a symlink to the components contained within the SSP application, and leave it hidden within the `/Applications/` directory.<br>
You can see this in the SSP post-install script:

```bash
# Remove standalone Jamf Connect.app if it exists
APP="/Applications/Jamf Connect.app"
if [[ -d "$APP" ]]; then
  /usr/bin/logger "Deleting standalone Jamf Connect"
  rm -rf "$APP"
fi

# Add symlink to the new Jamf Connect.app to handle keychain access
ln -s "/Applications/Self Service+.app/Contents/MacOS/Jamf Connect.app" /Applications/

# Hide the Jamf Connect.app symlink
chflags -h -P hidden "$APP"
```
{: file='postinstall_2'}

This is useful to know, as up until that version, using the JC 2.X LaunchAgent was _probably_ fine, as the target path of `/Applications/Jamf Connect.app/` still existed, even though it was pointing at something new.

#### However...

With the release of SSP 2.15 and later, Jamf have now changed the post-install script so that it no longer creates that symlink:

```bash
# Remove standalone Jamf Connect.app if it exists
APP="/Applications/Jamf Connect.app"
if [[ -d "$APP" ]]; then
  /usr/bin/logger "Deleting standalone Jamf Connect"
  rm -rf "$APP"
fi

# Host App manifest handles symlink for jamfconnect_tool, so no need to create it here anymore

# Hide the Jamf Connect.app symlink
chflags -h -P hidden "$APP"
```
{: file='postinstall_1'}

### Where does this become a problem?

If you've already migrated to SSP, you know that it handles updates automagically, independently of Jamf Pro server upgrades, which SSC used to rely on.<br>
However, JCL doesn't do this.

I'll caveat the following information by saying that it's not really relevant if you're using automated methods of updating JCL, such as installomator, or Jamf Apps auto-patching _stuff_.

If you're manually updating JCL then you'd be forgiven for thinking you need to also update the LaunchAgent which continues to be included in the JC 3.X DMG, as this previously hasn't caused an issue.

_But it can do now._

If you have SSP installed, subsequently update the JCL version _and_ update the LaunchAgent with the content from the JCL DMG, you're going to break the SSP menu bar LaunchAgent as it's going to look for a file that doesn't exist.

This is the JC 2.X LaunchAgent content:
```xml
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">

<plist version="1.0">
    <dict>
        <key>KeepAlive</key>
        <true/>
        <key>Label</key>
        <string>com.jamf.connect</string>
        <key>LimitLoadToSessionType</key>
        <array>
            <string>Aqua</string>
        </array>
        <key>Program</key>
        <string>/Applications/Jamf Connect.app/Contents/MacOS/Jamf Connect</string>
        <key>RunAtLoad</key>
        <true/>
    </dict>
</plist>
```
{: file='com.jamf.connect.plist'}

This is the content from the LaunchAgent that comes bundled in SSP:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">

<plist version="1.0">
<dict>
	<key>KeepAlive</key>
	<true/>
	<key>Label</key>
	<string>com.jamf.connect</string>
	<key>LimitLoadToSessionType</key>
	<array>
		<string>Aqua</string>
	</array>
	<key>Program</key>
	<string>/Applications/Self Service+.app/Contents/MacOS/Jamf Connect.app/Contents/MacOS/Jamf Connect</string>
	<key>RunAtLoad</key>
	<true/>
</dict>
</plist>
```
{: file='com.jamf.connect.plist'}

Whilst both files have the same name, the content points to a program at a different file path.<br>

## Final thoughts...

The change from Jamf Connect of old, to Account Management integrated into Jamf Self Service+ is only a small component of the SSP migration journey.<br>
There are many resources online that cover the wider migration journey, and other components that aren't included here.

Personally, I don't feel that Jamf have communicated the change to the LaunchAgent components loudly enough for admins to have understood these nuances from day one. <br>Continuing to include the old LaunchAgent within the JC 3.X DMG is also confusing, as without digging deeper it's very easy to miss the new detail.

Hopefully this post helps some admins bridge that gap and continue to enjoy success with ~~Jamf Connect~~ macOS Account Management with Self Service+ functionality.




## TL;DR

If you've migrated to Jamf Self Service+, ___do not___ re-deploy the LaunchAgent contained within the `Resources` directory in the Jamf Connect 3.X DMG.

You'll break your deployment!




<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
>**Disclaimer**<br>This post is designed as an informative explanation and not designed to claim any credit for software design or creation.<br>Content within this post that is not my original creation remains property of the original creator.
{: .prompt-info }

<!-- markdownlint-restore -->
