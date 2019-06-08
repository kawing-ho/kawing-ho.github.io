---
layout: post
date: 2019-06-08
title: Pseudo Incident Response
categories: ['sec']
---

# TL;DR

I wrote this just to shed some light on what malicious operations are out there, what thought processes this actor was having, and the tactics/technologies being employed by both sides in attacking/countering. After all security is a somewhat _cat-and-mouse_ scenario :rat::cat2:  

<br/>

# So... what happened?

Well my friend on Facebook executed some dodgy Javascript that they shouldn't have --- triggering a CSRF on _Facebook_/_Messenger_ to spam his friends list <small style="font-size:14px;color:grey">(including me)</small> with a very interesting message:

![no_phish_pls](/assets/images/pir-msg.png)
> Loose translation: <u>Hey! Is this your video?</u> :open_mouth: 

#### Some interesting observations:
- The script was able to detect his Facebook language settings
   - It would mimic the language for the most effect  
     (compared to just sticking to English)
      - Clearly their target(s) are not restricted to any country.
      - Its a drag-net style phish, aiming to hit anyone and anything :fish:
   - The downside to this is if you get the language wrong, it raises suspicion and kills the phish attempt altogether...
- The message was also tailored with the profile picture of the victim
   - Really nice touch to make it more personal
   - Social engineering at play, enticing victims to click
   - It really targets people's insecurities
      - we Sec people tend to forget, not everyone is good at OpSec :wink:  


<br/>

# Sandboxing time

Here's the URL in question:
```perl
https://biscuit-citron-quasar-c6os.glitch.me/#J3wAVJ6IIYF7YG30W4SVxko7Jj6ze%2Fm4lwcHO00oY7Oh1fg4YjHN%2B9Lb9sUEq3T2UoUwpf30WxbdG1C8ys1nyv%2B0hWzGF9ua%2B6baC9x0QkuZpton6sa1kXT17hZKyAAv%2Fqe1UeL0d8URKFd8J8R4tZCOMYbAy%2FhXq9af47CQ2bLIEdNGqFwe%2BZNKT0Mw8KtfdV5lBKRCkXWtVD6bjFVvNeWSYYGGP3RuFGrBfiwzYKkcnPPjvjD7Crz7yMpgJEIZbA4IAJCK%2BpfbtwCnXB2i6WnoBynw%2FA4xYhm5ue8%2Fob5Zas3rWxJuTOE0pvu1Xa17O4HlNCjJoTFnm8bMTYO%2FOA%3D%3D
```

Not wanting to risk executing any Javascript locally, and also not wanting to spin up a VM -- I visited the link with [`browserling`](https://www.browserling.com){:target="_blank"}

<br/>

# Jekyll and Hyde

> Unfortunately at the time of writing I wasn't able to retrieve the source, and I didn't save it before the malicious app got removed :cry:  

### The Platform

Did some digging around the URL and discovered that the app was being served on a legitimate platform known as [**Glitch**](https://glitch.com){:target="_blank"}

It's a less technical version of **`GitHub`** as far as I can tell, it promotes code-sharing and collaboration, but edits are similar to spreadsheets and not commit-based.  

The main point here though is that no accounts are needed to store and host code. The [malicious actor](https://glitch.com/user/4869718){:target="_blank"} created multiple instances of their app without needing to leave any information behind. :feet:

![anon](/assets/images/pir-anon.png)
<br/>


### The App

The harmless app that loads initially is a weather forecast widget on a grey backround. Thats about it...

<br/>

### The Hidden JavaScript

The hidden JS is either obfuscated or pulled in from a remote server. Without doing any reversing or fully reading the code, it:
- Checks what browser version you're using
   - With Sandbox using IE, it doesnt do anything malicious
   - With Firefox and Chrome versions ~65 or over, it does dodgy stuff
- Sends a POST request with a token to <small style="color:red">`weatherapi.pro`</small>  
  (I think its the same one as the commented section in the URL)
  - Users not knowledgable about domains/TLDs might not see the issue
  - But most of us would be slightly aware of `.pro` wouldn't we? 
- Makes additional requests to:
    - `xxx.dtscout.com`
    - `xxx.eyeota.com`
  - These are seemingly used for analytics
  - They are actually tracking how many and where people clicked from! 
  - Perhaps to log IPs for future attacks? 
    - After all they do not have access to the logs on **Glitch**
- Finally --- serves up a generic phishing page
        ![fakefb](/assets/images/pir-fakefb.png)
        
<br/>

### The control server

Using `whois` it shows that [`weatherapi.pro`](https://www.robtex.com/dns-lookup/weatherapi.pro){:target="_blank"} is served by     
**`NS-CLOUD-E*.GOOGLEDOMAINS.COM`**... probably a Google Cloud Compute instance?

The domain registrar is NameCheap, the domain itself was purchased just **<u>6</u>** days before the trap was activated :pencil:  

![spammonkey](/assets/images/pir-spam.png)

When visiting, a simple `GET` request only returns `{"status":"200"}`, which might fool most people, but I believe they're relying on some obscure system of tokens and passwords in the backend to determine what content to return or to _"play dead"_.


<br/>

___

# Update: 8th June

So I checked back today and guess what?
![rip](/assets/images/pir-rip.png)
![rip](/assets/images/pir-suspended.png)

Kudos to the Glitch people for quickly responding to either an internal check or user reports :space_invader:

<br/>

The webserver at `weatherapi.pro` was also taken offline, the malicious actor is probably cleaning up their tracks or having their resources terminated by Google :smile:  

<br/>

___

# Final words


I approached with caution initially because I thought it was a server that  
   - delivered drive-by downloads for malware
   - served more malicious JS  

In the end it was simply a complex but somewhat effective phishing setup.

<br/>

To make it better they could have also customized the phishing page to follow the language settings of the victim, consistency is key for convincing victims that your page is legit :sparkles:  

I found it interesting that they bothered to engage with analytics in their attacks as well, although I've been told countless times that attackers can be as professional as well-oiled developer teams as well :100:

> It's sad for me to say this but I know people who are quite likely to fall for this, although it's glaringly obvious for Sec people...  
