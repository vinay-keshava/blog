---
title: "Software Freedom Camp 2021 Snikket"
date: 2022-03-29T20:01:24+05:30
weight: 10
description: "Learnings from software freedom camp"
tags: ["git.fosscommunity.in", "debian", "fsci"]
type: post
---


Back to writing after a 2-month long Semester End Exam !!!!.  
Everyone uses messaging platforms like WhatsApp, Facebook, Signal, Telegram, and various other applications to communicate with people, in this blog post I would like to introduce to XMPP Protocol( Extensible Messaging and Presence Protocol), **XMPP** is an open, decentralized universal messaging standard for instant messaging, voice/video calls.   Many Applications or Clients are built using the XMPP protocol due to its open nature. Platforms like WhatsApp, Telegram, Signal impose vendor lock-in wherein the user using the product or service cannot transit to the competitor’s product/service. To overcome vendor lock-in issues and privacy issues one of the minimal, simple best solutions is to set up a Snikket server of your own, where you can own your data.

# What is Snikket

[Snikket](https://snikket.org/) is a simple, customized messaging platform that is different from other messaging apps like Whatsapp, Telegram. Snikket is a decentralized messaging platform which means anyone can host their snikket server on their cloud, it allows everyone to host their server.      
Snikket is free software, a privacy-friendly messaging platform based on XMPP protocol, it can be self-hosted by anyone. Snikket provides an android application client to connect to any XMPP servers and using a snikket account you choose any XMPP clients if you want to connect using android applications like monocles chat, blabber, Snikket app other desktop clients recommended are dino-im and gajim.        

# Experience of Running Snikket Server 
Snikket is my first self-hosted service, before talking about the experience of running the snikket server I would like to talk about [Software Freedom camp 2021 ](https://camp.fsci.in/),the camp is organized by the Free Software Community of India. As a part of the camp initially, learners were made to understand the philosophy and intention behind free software. Later during the project phase, learners were allowed to choose certain available topics proposed by the mentor.    
After joining the sfcamp as a learner, I choose to learn system administration and Debian packaging, under system administration one of the deliverables was to set up a Snikket server of my own.  
To run any snikket server basic requirements are a domain name and a VPS (Virtual Private Server )to run your snikket server. I had signed up for the Github Student Developer pack through the pack got the free domain from name.com and I choose Amazon Web Services Free Tier as my VPS.
```
  Snikket is all about the
        DNS
        Docker
        Daemons
```
Snikket also has an option of creating circles, limiting users to that circle only, although any user can talk to any individual by providing an XMPP or Snikket username. Snikket uses an invite-based procedure for account creation on the server. Only the admin can have the authority to create an invite link. Snikket also supports audio and video calls of great speed, these are some of the [features](https://snikket.org/app/features/) of the snikket application. I have been using Snikket and invited most of my friends to my Snikket server. Initially, it took time to make them understand how it is completely different from other messaging platforms. Later educated them about how decentralization, vendor lock-in works and then introduced them to Snikket and other XMPP-based clients.  
The learning while setting up the snikket server was got introduced to Docker Container, Domain Name System( DNS ), how server logs are checked, and debugging the errors.

# Guide to Setup Snikket Server
Here is the quickstart guide to setup the [snikket server ](https://snikket.org/service/quickstart/), this is the official guide by Snikket to setup the snikket server .
The detailed documentation is [here](https://github.com/snikket-im/snikket-server/tree/master/docs).
Snikket requires few ports to be open for communication refer [this ](https://github.com/snikket-im/snikket-server/blob/master/docs/advanced/firewall.md) which clearly mentions the firewall rules and ports required.
Snikket Source code [here](https://github.com/snikket-im).



Thanks to [Ravish](https://ravish0007.github.io) and [Sahil](https://blog.sahilister.in) for helping to set up the Snikket server.


<h1>Next Goals</h1> - Reverse Proxy by Nginx .


Bye for now .   
```
    :wq
```

