---
layout: post
title:  "A weapon to surpass Metal Gear"
date:   2021-02-06 5:55:55 -0500
categories: janitoria macos
tags: applescript projects dialumberjack
---
I've been pondering the issue I [wrote about](https://samess-flowers.github.io/tech/macos/2021/02/03/assembling-and-building.html) the other day, and I think I've settled on a solution.  I'm going to build a program to tie together all of the disparate scripts and subsystems currently in my Jamf environment.  A common UI to tie everything together and present the same face to the user.  Because it handles dialogs and I am physical incapable of restraint, I have named it [DiaLumberjack](https://github.com/saess-sep/dialumberjack)!

### Guiding principles
The North star for this project is consistency.  No matter what systems I end up integrated, I want my C3UPs to have the same experience, with the same language, same UI, and same behavior.  I want the system to have a lot of flexibility, so it can be used not just for updates, but for any system administration task.  I want the system to require as little infrastructure in place as possible; the initial prototype is going to use Jamf to run the script, shell scripting (Sh/Bash/Zsh) for the logic, and AppleScript to generate the UI.  AppleScript gives more options than JamfHelper and will make the system easier to port in case someone wants to use it with a different MDM.  AppleScript is available on all Macs out of the box and is a stable (AKA dead) language, so there won't be a lot of scrambling to patch when a new update comes out.

### Function
DiaLumberjack will function as follows:
1. The DiaLumberjack script is uploaded to Jamf Pro
2. DiaLumberjack is called in a policy, using the available parameters to determine what should be displayed and what options should be available.  A parameter also determines what policy is called after the dialog finishes
3. The script runs as its own policy with a schedule set in Jamf Pro
4. The user interacts with the dialog, either delaying to an acceptable time or allowing the action to take place.
5. Once the action is agreed to, then DiaLumberjack will call the Jamf policy that actually does the action in question

### Looking forward
I already have the AppleScript parts mocked out into a UI.  I'm currently building the shell script portion.  The script is currently limited to updates, but I fully intend to work out some clever system to allow extensibility to other tasks.  Maybe parameter 1 is the policy to call, parameter 2 is the type of task, and then everything else can change depending on parameter 2?  Something like that.