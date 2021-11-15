---
layout: post
date: 2021-07-14
categories: ['life']
published: true
title: Highest Payout So Far
---

## Background

> This is just a short post for me to count my victories as they come :)

Definitely a luck factor in finding this vulnerability:
- Target was a massive web application with over **500+** endpoints
- Hunted for 15 minutes, stumbled upon a file viewing functionality with full physical path in the query parameters 


### Not always straightforward
I wasn't able to leak easy-win files:
- Obvious blind-files such as `C:\win.ini` couldn't be read most likely due to lack of permissions
- Application source code such as `.aspx.cs` couldn't be read too as I couldn't figure out where they were stored on the underlying filesystem.

### web.config file to the rescue 

Did some looking around and managed to find `web.config` file to prove impact. This was only possible due to the full physical path being leaked as I could simply traverse up the path with `/web.config` at the end of each subdirectory.  

The `web.config` file contained:
- internal URLs to additional applications
- additional endpoints within the monolith application
- software version information and disclosure (low impact)

## Payout
![lfd_payout](/assets/images/lfd_payout.png)


## References

[web.config disclosure for impact](https://samcurry.net/reading-asp-secrets-for-17000/){:target="_blank"}
