---
layout: post
title:  "Installomator"
date:   2021-02-14 5:55:55 -0500
categories: janitoria macos
tags: macos installomator jamf
---
Installomator is a rad script from Armin Briegel of [Scripting OSX](https://scriptingosx.com/2020/05/introducing-installomator/).  It installs applications from the internet on your Mac with a single line on the Terminal.  It works by taking a download from a stable link, then checking it against the known developer information to ensure the download hasn't been tampered with, then unpackaging and installing the app.  It's available via [GitHub](https://github.com/scriptingosx/Installomator)

### As a script
As a script, Installomator is awesome, but of limited use.  It isn't a full package manager like Brew or Apt, and it only has a handful of applications to choose from.  The installation process requires Admin privileges and is only available from the command line.  On the bright side, it's a single line, it's easily configurable, and it always grabs the latest version from the developer, so there's no need to worry about getting a bad patch.  You simply clone the repo and then run `sudo ./installomator.sh visualstudiocode` or whatever thing you actually want to install.  The full list of labels is available [here](https://github.com/scriptingosx/Installomator/blob/dev/Labels.txt).

### As a policy
Where Installomator really shines is when used through a management platform like Jamf.  Installomator is expressly written with this in mind, so you can upload it as is to Jamf Pro without needing to mess with argument parsing or running as root.  You can even pass configuration options as parameters so you only need to have one script in Jamf.  I use Installomator to offer Self Service installation options and to patch third party apps.  Both of those are done via a script payload in a policy, but patching has a few extra steps.  I'll  be talking about the patching in a later post.

#### Self service policy
To start out, get the text of Installomator.sh into Jamf.  The "correct" way would involve cloning the Git repo, but you can honestly just copy and paste it since Jamf has a text entry field rather than a file upload button for scripts.  Once it's uploaded, head over to the Options tab and add in the config options you'll be playing with most frequently.  
![Installomator options](/assets/installomator/installomator-script-options.png)
I have `DEBUG` in the list, but honestly you should just leave that at 0 all the time.  `BLOCKING_PROCESS_OPTIONS` and `NOTIFY` are both good options to have in the list.  `SADNESS_TIMER` is specific to [my fork of Installomator](https://github.com/saess-sep/Installomator), and is a value for a `giving up after` Applescript option on the dialogs (since some users would just have the prompt open forever, stalling Jamf).  `prompt_user_instakill` is also specific to my fork, and is used to immediately kill apps like Dropbox that don't like closing when asked politely.

Once you have the script properly set up, you can start making policies.  The core of the policy is adding a script payload with a properly formatted label as an argument.  You can leave the configuration options as they are in the script or customize them as you'd like by adding them in as additional arguments using the format `CONFIG_OPTION_NAME=chosen_config_option`, taking care not to have any spaces anywhere in the command.  You also need to have all arguments in Jamf be contiguous, meaning no gaps between options from the top to bottom.

![A policy with gaps](/assets/installomator/policy-with-gaps.png)
This policy will not execute with any option after the label

![A policy without gaps](/assets/installomator/policy-without-gaps.png)
While this one will

Theoretically, you can put the arguments in any order, but for maximum readability, make sure that you have your arguments ordered from most common to least common when setting up the script.

After adding in the script, I like to have the policy also run recon to update the listing of what apps a computer has installed, but that's optional.  You then need to set up the self service tab to allow users to select it, and then scope it to the appropriate users, computers, and/or groups.
