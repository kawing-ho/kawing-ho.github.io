---
layout: post
date: 2025-07-01
categories: ['work', 'sec']
published: true
title: Internal Dump 3
---

## Background

Just noting down some neat tips and tricks learnt from a fellow Senior Consultant colleague who loves red team-y tradecraft :smile:

## Scheduled Tasks RDP

You may enounter cases where you have local admin on a machine and a high priv user such as a DA is logged in on the same machine. Trying to dump hashes is a big problem when there is EDR on the machine.

Instead, what my friend suggested was if they are connected via RDP, youc an just hijack their RDP session and run a scheduled task under their user context, something as simple as adding a new DA user.

This way you don't have to meddle with dumping hashes and still achieve desired escalation of privileges.

## VPN Tips & Tricks

### Download the installer

Most SSL VPNs have a web-browser based login which you need to auth to (hello MFA) and then access internal environments. If they haven't been configured correctly, you may be able to:

1. Download the installer directly (which is usually hosted in a well-known location)

2. Login using valid credentials without the hard-client prompting for further MFA.

### GlobalProtect: Portal auth vs Gateway auth

I believe he will be making a talk about this sometime soon, but basically portal auth sits above the gateway auth layer and you login there (hello MFA) to prove you can use the VPN, whereas gateway auth is used simply to authenticate to an underlying DMZ/destination server. This means that if you have valid domain credentials, you can just append `/gateway/` to the VPN path, and gain routes to internal network that way. 

## Check Tools

This is a discovery I made since late last year but still holds true in 2025, you should always use any `impacket` tools or adjacent tools such as `certipy` etc. that rely on those Python packages/libraries with a grain of salt.  

In my experience, sometimes using older versions of the same tool can lead to more accurate results/more stable execution outcomes. I think because the tools are open-source and are updated regularly its a fact of life that things will break from time-to-time.  

Basically you shuld aim to have multiple different Python virtualenvs, holding different versions of `impacket`, and if a tool is being weird, try it on different versions (or even forks) of it to see if it improves the situation. Don't just assume that tool no work = vuln no work...  
