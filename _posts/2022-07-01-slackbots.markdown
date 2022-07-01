---
layout: post
title:  "Slackbots"
date:   2021-07-01 5:55:55 -0500
categories: "Tutorial"
tags: "API","Slack","Python"
---
## To Slackbot or not to Slackbot
As far as I can tell, I probably don't need to be writing my own Slackbots very frequently.  Slack Workflows are a code-free option that automates Slack without having to worry about secrets management or write error handling code or anything of the sort.  Workflows should cover 80% of what we need with 20% of the effort of making a Slackbot.

**However**, Slackbots allow much more complex behavior, greater flexibility, and the ability to query the Slack Web API.  I'm going to pull two terms out of thin air: interactive and non-interactive Slackbots.  Interactive Slackbots are the normal bots you're familiar with: you say `[[millstone]]` and it tells you about [Millstone](https://scryfall.com/card/m14/213/millstone).  They listen for events and respond, usually in Slack.  They require some kind of web server to handle requests.  Non-interactive Slackbots are basically a set of permissions.  A non-interactive Slackbot will get installed in Slack the same way, but it "doesn't do anything".  Really, it doesn't do anything in Slack.  An example would be a script that makes Slack API calls to look up a user's information using a bot token.  This is nice because it doesn't require a web server and you don't need to design a user interface.

## Set-up
This is a brief tutorial to walk you start to finish from having no code to have a working Slackbot that responds to @mentions with the :eyes: reaction in whatever channel you invite it to.  Along the way we'll get the code and config into source control, without getting our secrets in there.

### Repo
So far, all the Slackbots I've written have been in Python using the Bolt SDK from Slack.  You can do a pretty quick setup by running these commands, which makes a folder, installs the pre-requisites for a basic Slackbot, and then begins source controlling the folder.

```
mkdir slackbot && cd slackbot
python3 -m venv .venv #this creates a Virtual Environment to throw all your packages into
source .venv/bin/activate #this activates the venv and will start using the python interpreter with your packages
pip install --upgrade pip #this upgrades the version of pip installed in your venv
pip install requests slack_bolt slack_sdk python_dotenv
pip freeze > requirements.txt #make a requirements file so that you can reinstall quickly from source control
echo ".venv/" >> .gitignore #prevent virtual environment files from getting sucked into source control
echo ".env" >> .gitignore # prevent your dotfiles WHICH MIGHT HAVE SECRETS IN THEM from getting sucked into source control, which can be a career modifying event.  DON'T PUT SECRETS IN SOURCE CONTROL PLEASE
touch app.py
git init
git add .gitignore requirements.txt app.py
git commit -m "Initial Commit"
```
If you're using Pylance with VS Code, hit Command+Shift+P, type in "Python: Select Interpreter", and choose the python interpreter in your Virtual Environment.  That will keep if from complaining about missing imports.

### Slack App
An app also needs to exist in a Slack workspace.  If you're reading this article, you don't want your Slackbot to be in a production Slack.  Again: do not make a test app in a production Slack instance.  Go to https://api.slack.com/apps and then select "Create New App".  There are two options, "From Scratch" and "From an app manifest".  App manifests are a configuration file containing all of your Slackbot settings in one file (that can be source controlled :roll-safe:), but let's start with "From Scratch" for now.

You'll want to find these settings and enable/set them:

* Socket Mode → Enable Socket Mode
    - This will ask you to name a token
* OAuth and Permissions → Bot Token Scopes → `reactions:write`
* Event Subscriptions → Enable Events
* Event Subscriptions → Subscribe to bot events → `app_mention`
* Install App → Install App to Workspace

Then, go to App Manifest.  This will show you the App Manifest that reflects all of the settings you'd need.  You can export that and save it in your project folder to get it into source control.  Don't forget to `git add manifest.yaml` and `git commit`!

## Scoping Things
The eagle eyed among you may have noticed that we added a bunch of settings and I didn't explain them at all.  Let's walk through them now.

Socket mode is an alternate method of connecting to Slack.  The normal method of Request URLs involves having an HTTPS endpoint on the public Internet that is listening for messages from Slack.  <rant>This is a whole lot, requiring a server, a public IP address, public DNS, an ungodly amount of maintenance to not have your box get pwned, and other gross things.  Instead, socket mode uses a token to create a tunnel between your web server and Slack that data can pass through without getting a lot of requests to join really cool botnets from countries ITAR doesn't want you talking to.</rant>  Socket mode allows you to send Slack a token and then open a secure tunnel to pass communication back and forth.

OAuth permissions are the very granular permissions that we can assign to our Slackbot to allow it to see and do things.  The Slack API will list the permissions you need to do any given task in the page for the API call you want to use.  In our case, we're using `reactions:write` and `app_mentions:read` to be able to see when people @mention our app and to be able to react to messages.

To be able to respond to App Mentions, we need to set our app to listen for them.  We do this with an Event Subscription for App Mentions.  This requires the `app_mentions:read` permission.  If we don't already have that permission set, Slack will automatically request it for us when we add that subscription.

Finally, we need to install our app.  Whenever an app is created or its permissions change, it must be (re)installed.  This can require some administrative approval, but for a non-prod test instance, you can probably just approve it yourself.

## Secrets
Now that we have a Slack app, we also have some secrets!  Specifically, we have an App Token and a Bot Token.  The App Token is used to set up the socket for communication with Slack.  The Bot Token is used to authenticate for actual calls – listening for events, posting messages, etc.  For our purposes, we're going to store our secrets in a `.env` file in our project folder.  We already added this file to our `.gitignore` file (which tells git which things not to track in source control) during the first step.  Now we have to actually make the file and populate it with our tokens.  First, let's find our tokens!  The App Token is available on the "Basic Information" page, under "App Level Tokens".  Click on the name you came up earlier to find your `xapp-` token (sometimes called a "zap"). 

For your Bot Token, go to "OAuth & Permissions" and find it under OAuth Tokens for Your Workspace.  It will start with `xoxb-`.

In the project folder, create a new blank file called `.env` and put the following information in it:

```
#SLACK TOKENS
SLACK_APP_TOKEN = "xapp-###-###-###"
SLACK_BOT_TOKEN = "xoxb-###-###-###"
```

## Writing the actual app


So, we've done a lot of things, but we haven't written any code.  Don't worry, we'll start that now. A very simple Slack App will look something like this:

```
from slack_bolt import App
from slack_bolt.adapter.socket_mode import SocketModeHandler
from dotenv import load_dotenv
import os
 
load_dotenv()
SLACK_BOT_TOKEN = os.environ['SLACK_BOT_TOKEN']
SLACK_APP_TOKEN = os.environ['SLACK_APP_TOKEN']
 
app = App(token=SLACK_BOT_TOKEN)
 
@app.event("app_mention")
def mention_handler(event, say):
    timestamp = event["ts"]
    channel = event["channel"]
    reaction = "eyes"
    app.client.reactions_add(name=reaction,channel=channel,timestamp=timestamp)
 
if __name__ == "__main__":
    handler = SocketModeHandler(app, SLACK_APP_TOKEN)
    handler.start()
```

What does that do?  Let's walk through it bit by bit

```
from slack_bolt import App
from slack_bolt.adapter.socket_mode import SocketModeHandler
from dotenv import load_dotenv
import os
```

First, we import several modules.  The `from MODULE import THING` allows us to only load one thing from a module and also means that we don't have to prepend the module name.

```
load_dotenv()
SLACK_BOT_TOKEN = os.environ['SLACK_BOT_TOKEN']
SLACK_APP_TOKEN = os.environ['SLACK_APP_TOKEN']
This runs a function from the dotenv module that allows us to import variables from a `.env` file in our project folder as if they were environment variables.  This is a decent way to store secrets, but in production, you really don't want them stored in plaintext on disk.
```

Then, two variables are created by reading from the "Environment Variables" we've presented.


```
app = App(token=SLACK_BOT_TOKEN)
 
@app.event("app_mention")
def mention_handler(event, say):
    timestamp = event["ts"]
    channel = event["channel"]
    reaction = "eyes"
    app.client.reactions_add(name=reaction,channel=channel,timestamp=timestamp)
```
This is the actual "app" part of our App.  First, we instantiate the `app` object by calling App from `slack_bolt` with our Bot Token as an argument.  This makes a magic Python object called `app` that applicates all of our Slacks.  It has most of the code we want to actually use the SDK located within it.

Next we define a function with the `app.event("app_mention")` decorator.  Decorators are a way of inserting extra goodness into function or class in Python.  In this case, we're adding some goodness from our app for handling events of the App Mention type.  We can get two "free" parameters in "event" and "say".  Event is a Slack object in JSON format that contains the response from the Slack API and includes the timestamp, which is what slack uses to identify individual messages, and the channel, which is the channel the reaction was posted in.  We add in the string eyes and then we make a Slack web API call to add a reaction of :eyes: to the message that mentioned us in the channel where we were mentioned.  It will look like this:

```
if __name__ == "__main__":
    handler = SocketModeHandler(app, SLACK_APP_TOKEN)
    handler.start()
```
Okay, so this last bit includes Snake Magic, which I will explain elsewhere.  Inside the Snake Magic, we instantiate a SocketModeHandler object with our App object and App token and then start it.  This will open up a Bolt Server where we patiently await requests.```

## What have we learned today?
Hopefully, you now know how to make an interactive Python SlackBot.  We also touched on secret management and set up a fairly portable development environment with source control for our project.

If you're excited to try more with Slack bots, you could look into adding slash commands (don't forget to `ack()`), responses using `say()`, or adding in more advanced logic so that your Slackbot can respond to different people differently.


### Some links:

Twilio guide that talks about Socket Mode: https://www.twilio.com/blog/how-to-build-a-slackbot-in-socket-mode-with-python

Official Slack guide that doesn't talk about Socket Mode: https://github.com/slackapi/Slack-Python-Onboarding-Tutorial

Slack Bolt Reference: https://slack.dev/bolt-python/concepts

Slack API Reference: https://api.slack.com/docs

