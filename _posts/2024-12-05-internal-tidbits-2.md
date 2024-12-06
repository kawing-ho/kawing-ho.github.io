---
layout: post
date: 2024-12-05
categories: ['work', 'sec']
published: true
title: Internal Memory Dump 2
---

# Background

Just wrapped up two more internal pentest engagements and learnt a few niche things as well which will help me do pentests better-er in the future.  

---
   
# Tips & Tricks


## Impacket and Virtual Environments

`impacket` has been acting a bit fucky-wucky lately especially with `ntlmrelayx` and its multi-relay mode, therefore it is worth playing around with different versions of it within your tools:

```shell
# assuming you are using your preferred Python version
# assuming you already have virtualenv/venv installed
git clone https://github.com/fortra/impacket.git
cd impacket
git checkout db53482 # tagged as v0.12.0, change to other commit hash if needed

python3 -m virtualenv impacket-venv
source impacket-venv/bin/activate
pip install . # this will build impacket

# now run your tools and see the different impacket-header pop up
impacket-getTGT
deactivate
impacket-getTGT # should be back to normal after exiting the venv
```

## MMC Snap-Ins

- On Red/Purple Team engagements you can leverage MMC Snap-Ins as a low-priv user on an SOE environment to enumerate some good initial pieces of info:
    - Local users & groups
    - Shares & sessions
    - Certificates (can identify vulnerable templates from ESC1 up to ESC4)
    - Services & Scheduled Tasks for priv-esc/persistence
- This is considered quieter than using things like `schtask` or `net`
- [ref. link](https://www.ired.team/offensive-security/enumeration-and-discovery/enumerating-users-without-net-services-without-sc-and-scheduled-tasks-without-schtasks){:target="_blank"}

## DarkTrace

- I encountered DarkTrace on my recent engagement, and I can say it is extremely hostile towards suspicious traffic
- It can and will kill/reset network connections on the switch level to stop attacks from "compromised" devices on the network
- It can also detect suspicious traffic on the network, such as NTLM authentication being coerced from a Domain Controller! 
- I got detected just scanning one open port on another host in the network with `nmap` :joy:  
  
## More Sneaky Techniques

- ADSIsearch+LDAP Walking
- Excel
- Using RDP to log in to machines is slightly more "normal" than other command line logins. Although is less stealthy if you have to "kick" others off their sessions to do so...  
- Responder has a RespondTo configuration option to avoid causing network disurptions

