---
layout: post
title:  "Naming your computer"
date:   2021-02-15 5:55:55 -0500
categories: janitoria organization
tags: naming
---
I love naming computers!  It's one of my favorite parts of my job.  However, I think it's best for the person using the machine to name it.  I encourage my coworkers to name their own computer, and sometimes get asked how they should pick a name for their machines. This is primarily a guide for them, but hopefully it is useful to anyone reading it.

# What is a host name?
The host name is the human-readable name that your computer identifies itself internally (such as in a command line prompt) and externally on whatever networks it is connected to.  It's often tied into network systems such as Active Directory and allows other users and IT administrators to identify a specific machine and the things it does on the network.  They're deeply related to domain names, and in most cases your computer's host name is also its domain name.

## Picking a name
### Unique
A hostname needs to be unique to whatever network it will be on.  For an example, if I have a computer named "Kodiak" on my work network and one on my home network, that's perfectly fine, since the networks are on different domains and the full names would be "Kodiak.work.com" and "Kodiak.samess.local", but the best practice is to have any names you're using be unique to you and any networks you use, so my home computer could be named Kodiak and work computer could be named Panda.  This avoids confusion if you have to take your computer home or your personal computer into work.

## Avoid names
I would avoid using your name and especially avoid using other people's names for your host name.  It's awkward enough to get a message saying "Sam fell off of a counter and shattered", but it's even worse when you aren't Sam.

## How to check
If you're naming or re-naming a computer at work, it's a good idea to try to do an `nslookup` on your chosen name to see if anything else on the network is currently using it.  You might also try looking up names *similar* to your chosen name, such as "Phyrigia" and "Phyrigian".

### Short
In the olden days, host names were strictly limited to only a few characters.  Today, you can have an arbitrarily large host name, but it's a good idea to keep them short.  15 characters allows compatibility with most of the legacy protocols still in use, without being too short.  A short host name is also easier to work with.

### Easy to spell and say
A good host name is easy for *someone else* to spell and say.  It's great if you can spell it, but if someone else needs to connect to your computer or find it from a list, then you're probably going to end up spelling it out loud for them.  As fun as it is to have your computer be named "Vercingetorix" (Vehr-kin-get-or-icks), it's not the most helpful choice.

## No homophones
I learned this on the hard way: never use homophones.  I had a computer named "Serra" (after the Fire Emblem character), and everyone who was told to look for it would always look for "Sarah".  This also applies to intentional misspellings, which can be good for a joke but bad for a host name.

### Meaningful
A good hostname should be meaningful to you and make you happy when you see it.  It can be as simple as a reference to a book you like (such as "Wiggen") or a favorite game ("Midgar").  It can be something with a special meaning to you, a nice pun ("BigMac"), or just a cool sounding word ("Dauntless").  The important thing is that it means something, even if it's small.  That will help you remember it when you don't have it on hand to check.

### Inoffensive
This one is simple: since your computer's hostname is used for communication, you need to keep other people's feelings in mind when deciding on a name.  This depends on where you are and who's on your network, but use common sense and err on the side of caution.

# Wrapping up
Hopefully, this guide helped you come up with a good name for your new computer!  Remember, keep it short, simple, meaningful, and inoffensive.