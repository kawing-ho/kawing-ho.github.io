---
layout: post
title: My 90's Hacker Story
categories: ['job']
date: 2020-07-24
---

# TL;DR 

> - Senior finds webapp 0-day in old software
> - I find a developer fuckup
> - Our teamwork leads to us pwn'ing a client **HARD**
> - <u>PII leaks</u> / <u>Modifications to live data</u> / <u>Financial damage</u>
 
--- 

# Disclaimer

A lot of the text and images in this post will be <del>**heavily redacted**</del>  because:
- The software company which produced the software no longer exists
   - therefore bugs cannot be patched
- As far as I know, the client site is still running and may be vulnerable
- At least 1 other company has a website using this software
- My senior might want to hold on to this 0-day for CVEs etc...  

# Background

It was a chill Friday for me, I had no projects that day when my senior `@Osirys` approached me for help with a vuln he found.
I was a bit reluctant at first as I had other plans but decided eh why not! :smile:  

# The 0-day

The web application didn't seem very interesting at first, just a weird welcome/login page with a 90's look that would tell you to fuck off if you entered invalid credentials.

![story-login](/assets/images/story-login.png)

### Discovery

While directory brute-forcing, he discovered a `/tools` directory!
![tools](/assets/images/story-tools.png)

This led him to a completely new page, which included Documentation and <u>Examples/Samples</u> of certain things...  
  
![tools-index](/assets/images/story-tools-index.png)

### Initial exploitation

By browsing around, there were a couple of **Samples** to play around with, one of them was an email subscription functionality.  
What was important was the <u>bottom of the page</u>:  

![email](/assets/images/story-email.png)
![sauce](/assets/images/story-view-source.png)

> I guess the developers in early 2000's didn't know the dangers of **`Local File Disclosure`** huh :wink:  

From there, he used the **`LFD`** to look at local files ---   
(even finding the config file for the webapp, sadly no creds were stored)
![cfg](/assets/images/story-cfg.png)
![hosts](/assets/images/story-hosts.png)

---

# Enumeration for fun and profit

### What I did
This is where my senior asked me to jump in and help:

I noticed the naming convention of certain backend files were `XXX<word>.xxx`  
Using the [big.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/big.txt){:target="_blank"} and Burp Intruder, I ran a bruteforcer for files on the filesystem and quite a number of them came back with a <small style="color:green">**200**</small> result!  
```
http://some.domain.com/xxx/XXXXXXbase.xxxxxx
http://some.domain.com/xxx/XXXXXXBase.xxxxxx
http://some.domain.com/xxx/XXXXXXLogin.xxxxxx
http://some.domain.com/xxx/XXXXXXaca.xxxxxx
http://some.domain.com/xxx/XXXXXXbase.xxxxxx
http://some.domain.com/xxx/XXXXXXbmp.xxxxxx
http://some.domain.com/xxx/XXXXXXcalc.xxxxxx
http://some.domain.com/xxx/XXXXXXcam.xxxxxx
http://some.domain.com/xxx/XXXXXXcat.xxxxxx
http://some.domain.com/xxx/XXXXXXcfm.xxxxxx
http://some.domain.com/xxx/XXXXXXchm.xxxxxx
http://some.domain.com/xxx/XXXXXXcmt.xxxxxx
http://some.domain.com/xxx/XXXXXXcom4.xxxxxx
http://some.domain.com/xxx/XXXXXXcom3.xxxxxx
http://some.domain.com/xxx/XXXXXXcom2.xxxxxx
http://some.domain.com/xxx/XXXXXXcom.xxxxxx
http://some.domain.com/xxx/XXXXXXcre.xxxxxx
http://some.domain.com/xxx/XXXXXXcrc.xxxxxx
http://some.domain.com/xxx/XXXXXXcrm.xxxxxx
http://some.domain.com/xxx/XXXXXXcrl.xxxxxx
http://some.domain.com/xxx/XXXXXXcrs.xxxxxx
http://some.domain.com/xxx/XXXXXXcrp.xxxxxx
http://some.domain.com/xxx/XXXXXXday.xxxxxx
http://some.domain.com/xxx/XXXXXXddl.xxxxxx
http://some.domain.com/xxx/XXXXXXdep.xxxxxx
http://some.domain.com/xxx/XXXXXXenq.xxxxxx
http://some.domain.com/xxx/XXXXXXfds.xxxxxx
http://some.domain.com/xxx/XXXXXXfd.xxxxxx
http://some.domain.com/xxx/XXXXXXfrm.xxxxxx
http://some.domain.com/xxx/XXXXXXfsm.xxxxxx
http://some.domain.com/xxx/XXXXXXias.xxxxxx
http://some.domain.com/xxx/XXXXXXinv.xxxxxx
http://some.domain.com/xxx/XXXXXXlogin.xxxxxx   [!]
http://some.domain.com/xxx/XXXXXXmenu.xxxxxx    [!]
http://some.domain.com/xxx/XXXXXXmlm.xxxxxx
http://some.domain.com/xxx/XXXXXXmsg.xxxxxx
http://some.domain.com/xxx/XXXXXXopen.xxxxxx
http://some.domain.com/xxx/XXXXXXops.xxxxxx
http://some.domain.com/xxx/XXXXXXprd.xxxxxx
http://some.domain.com/xxx/XXXXXXpri.xxxxxx
http://some.domain.com/xxx/XXXXXXprp.xxxxxx
http://some.domain.com/xxx/XXXXXXpw.xxxxxx
http://some.domain.com/xxx/XXXXXXrel.xxxxxx
http://some.domain.com/xxx/XXXXXXrom.xxxxxx
http://some.domain.com/xxx/XXXXXXron.xxxxxx
http://some.domain.com/xxx/XXXXXXrpt.xxxxxx
http://some.domain.com/xxx/XXXXXXsam.xxxxxx
http://some.domain.com/xxx/XXXXXXsem2.xxxxxx
http://some.domain.com/xxx/XXXXXXsem.xxxxxx
http://some.domain.com/xxx/XXXXXXsim.xxxxxx
http://some.domain.com/xxx/XXXXXXsorry.xxxxxx
```

### Developer Banter

Not sure if this was the original devs or the client's software devs but they have some really interesting UX choices!
![banter](/assets/images/story-banter.png)

### Interesting authentication

Most files which require authentication would have this at the top of file:
![pword](/assets/images/story-pword.png)  
This led to the discovery that the webapp authenticates on a page-by-page basis using _*gasp*_ username and password in the query string  :scream:  

---

# It can't be that easy... right?

After a few hours of combing through files, my senior had previously given a list of usernames that was found when performing account enumeration through inconsistent login messages. 
> The webapp would give a different response when a username was valid but password was incorrect :bulb:  

```
b****
b****
d*****
india  [!]
j***
j****
l****
l***
l******
p****
p****
s****
s*****
t***
```

When reading the source of the `XXXXXXmenu.xxx` file above,  
I realised the username **`INDIA`** <small style="font-size:0.7em; color:gray">(not actual username)</small> was hardcoded here... :thinking:  
![hardcode](/assets/images/story-hardcode.png)

From there, he suggested I try this username on the earlier discovered page to see what happens...
### It Was

Whelp, using the following URL:
```perl
https://some.domain.com/XXX/XXXXXXXXmenu.xxx?userid=INDIA&pword=<anything>
```
will grant you **<u><small style="color:red">FULL</small></u>** access to the internal menu that the system uses:
![menu-revealed](/assets/images/story-menu.png)

---

# We're In

It was bad enough that a user was hardcoded in, but this was a **<u>superuser</u>** which meant that we could do anything we wanted to maintain persistence:
- Add new users
- Change password of the current user
- Delete/Lock all other user accounts  
![superuser](/assets/images/story-superuser.png)

### Impact?

Possible scenarioes a real attacker could have done:  
<small style="font-size:0.7em;color:gray">(theres tons more but this is just the surface)</small>

- Invoiced a whole bunch of client companies leading to financial losses
- Leaked all PII of any employees found in the dashboard
- Leaked all confidential data regarding the multitude of services offered
- Cancel or deliberately edit routes to disrupt services
- Use internal mail system and phish users to pivot to other systems

![post](/assets/images/story-post.png)
![client](/assets/images/story-listing.png)

---

# Conclusion

That night, my senior emailed the client who then went to confirm the issue:  
![chaos](https://media1.tenor.com/images/a793b0e07f5860b304ccc7e3a9a10b1b/tenor.gif)![office](/assets/images/story-office.gif)  
They later confirmed that it was a third-party site which was vital to their operations!!! :scream::rotating_light:
  
  
  
  
Within a few hours, the login page was now replaced with this message:
![tempfix](/assets/images/story-disable.png)
which gives me confidence that they are working on migrating away from this garbage fire application once and for all. :fire:  

> I'm super happy I helped out because this finding was the most critical in the final report as well and I had helped in discovering it :smile:  
