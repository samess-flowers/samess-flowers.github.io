---
layout: post
title:  "Installomator"
date:   2021-02-21 5:55:55 -0500
categories: janitoria macos
tags: macos installomator jamf
---
Installomator is a rad script from Armin Briegel of [Scripting OSX](https://scriptingosx.com/2020/05/introducing-installomator/).  It installs applications from the internet on your Mac with a single line on the Terminal.  It works by taking a download from a stable link, then checking it against the known developer information to ensure the download hasn't been tampered with, then unpackaging and installing the app.  It's available via [GitHub](https://github.com/scriptingosx/Installomator).

This post is going to briefly show three different ways of using Installomator to install Visual Studio Code, but the same rules would apply for any of the other applications with [Installomator labels](https://github.com/scriptingosx/Installomator/blob/dev/Labels.txt).

# Locally
The simplest way of using Installomator would be downloading the script and running it locally.  To do this, you'd just clone the git repo, open a terminal window within cloned repository, and run `sudo ./Installomator.sh DEBUG=0 visualstudiocode`.  Let's break down the syntax on that command:
- `sudo` - Installomator requires being run as root, so you need to preface the command with `sudo`.
- `./Installomator.sh` - Execute the file named Installomator.sh in the current directory.
- `DEBUG=0` - One neat feature of Installomator is that it will accept configuration options as arguments.  You just need to be sure to type them correctly and without spaces.  By default, the `DEBUG` option is set at `1`, which will cause the script to do a mock run, without downloading or installing.  This sets it to `0`, to actually install VS Code.
- `visualstudiocode` - Each installomator label is handcrafted with love and listed in the script itself as well as in the Labels.txt file.  Normally, the label is the full name of the program without space, dashes, or underscores, so it's `visualstudiocode` not `vscode` or `visual-studio-code`.

Let's look at the output from that command:

![Installomator script output](/assets/installomator/script-output.png)

You can see that there's a good amount of helpful information output to the terminal (as well as to /var/log/installomator.log) showing progress and any possible errors that might have occurred.  If you already had VS Code installed and running, you would notice this popup box asking if you'd like to close VS Code:

![Installomator pop-up prompt](/assets/installomator/prompt.png)

If you answer "Not Now", the script will fail.  If you answer "Quit and Update", Installomator will send a "polite" close command to the program in question (which Installomator refers to as **blocking processes**).  Most apps will respond correctly to the polite close command, and ask you to save your work before closing.  Some apps, such as Dropbox and MS Team, will not close politely, and need to be killed instead.  There's a configuration option (`BLOCKING_PROCESS_ACTION`) for what to do with blocking processes that you can set accordingly based on what app you're interacting with and what behavior you'd like.

Once any potential blocking processes are taken care of, then Installomator will download the file to a temporary directory, check the Team ID of the downloaded file against the known Team ID of the program (which is defined in the label), and if it matches, it extracts the program from whatever packaging or compression it's in (also defined in the label), and installs it to the /Applications/ directory, then deletes the temporary directory it made earlier.  After everything is done, a notification goes out reporting what version of the app has been successfully installed, unless the option `NOTIFY=silent` has been set, in which case there's no notification generated.

# Adding Installomator to Jamf
Using Installomator locally is great, but it doesn't scale well.  If everyone in your team, department, or company should have VS Code installed, then you can use Installomator together with a management tool like Jamf Pro to easily deploy it without needing to package the app or worry about installing a stale version.  To do this, you'll first need to upload the Installomator script to Jamf, then you'll need to create and scope a policy with that script in the payload.

You can upload scripts to Jamf by going to Global Settings > Computer Management > Scripts and clicking the "+ New" button.  Give this script a **Display Name**.  I use the format "ScriptName DateRetrieved", so that I know how long it's been since I last updated the script in question.  In this case, let's call it "Installomator 21-02-21".  Fill in **Category**, **Information**, and **Notes** appropriately.

![Installomator Script General Tab](/assets/installomator/jamf-script-general.png)

Next, go to the **Script** tab and paste in the contents of the script from your text editor.  While you're here, you can set some default configuration options.  I usually have the options set as:
* `DEBUG=0`
* `NOTIFY=success`
* `BLOCKING_PROCESS_ACTION=prompt_user`
which makes everything run smoothly by default.  However, since Jamf allows you to pass different arguments to scripts in different policies, these are just the defaults.

To help out with changing those options, go to the **Options** tab, and mark them like this

![Installomator Script Options Tab](/assets/installomator/jamf-script-options.png)

to allow easy modification from the policy view.  I have them in order of most used to least used.  This is because you can't have gaps between the parameters in Jamf, every parameter between the first and last used needs to have something in it.  In a pinch, you can put a configuration option in the "wrong" spot since they don't have a required order, but it is more legible to put the config option where you labelled it.  You shouldn't need to mess with the **Limitations** tab, since Installomator will check for the macOS version within the script itself.

# Creating an automatic policy
Now that you have the script created, you can use it in a policy.  Let's say that you want VS Code installed on every computer in the Engineering department.  Assuming you already have the Engineering department as an LDAP group and have properly integrated Jamf with your LDAP server, you'll just need to make a smart group for Engineers that do not have Visual Studio installed.  You can use a smart group with these conditions to reach those Engineers:

![Engineers with VS Code Smart Group](/assets/installomator/smart-group.png)

You can also just apply the policy to the Engineering department indiscriminately, since reinstalling an application in macOS usually causes few problems.  I would recommend against doing that, since you'd then need to worry about blocking processes, as the Engineers could have VS Code open while you were trying to install it.

After uploading your script and creating your smart group, you can make your policy.  Since this is based around a smart group, you can set the **Trigger** to *Recurring Check-in* and the **Execution Frequency** to *Ongoing*.  This will ensure that if an engineer uninstalls VS Code, it will reinstall automatically when their Mac next check in.  Depending on your real life requirements, you can tweak the **Trigger** and **Execution Frequency** to meet your needs.

![General section for Automatic Policy](/assets/installomator/automatic-general.png)

Next, select **Scripts** from the sidebar and add a script payload using the script you added earlier.  Fill in the parameters as needed, making sure to add in the `visualstudiocode` label.  Also make sure that **Priority** is set to *Before*, to allow recon to properly run after installation.

![Scripts section for Automatic Policy](/assets/installomator/automatic-script.png)

Next, select **Maintenance** from the sidebar.  The **Update Inventory** box should already be checked.  This will ensure that Jamf runs recon and updates the smart group to remove the computer that now has VS Code installed.  Otherwise, the policy will run continuously until the next time recon runs and updates the installed applications for that computer.

Finally, scope your policy to the smart group you created earlier.  In real life, I would normally scope this to a test computer group first, to make sure everything works well before scoping it to my coworkers.  In this case we can just go ahead and scope immediately.

![Scope for Automatic Policy](/assets/installomator/automatic-scope.png)

Click the **Save** button and you are now done creating a policy to automatically install the latest version of VS Code to everyone in Engineering who doesn't already have it installed!  Next, let's take a look at deploying an app to Self Service using Installomator and Jamf.

# Creating a Self Service policy
