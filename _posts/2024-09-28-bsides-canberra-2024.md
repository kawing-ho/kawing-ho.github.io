---
layout: post
date: 2024-09-28
categories: ['life']
published: true
title: Bsides Canberra 2024
---

## All about coincidences

- I somehow managed to end up on the seat next to `@flux` on the inbound flight to Canberra :joy::airplane:  
- Bumped into few uni coursemates who I haven't spoken to for almost 5 years, they are doing well in their own careers and I'm super happy for them!  

## Sekuro OffSec Trip 2!

The Sekuro OffSec Team terrorizes Canberra and each other (with stickers) in the second installment of this ongoing saga. It was good to meet up with colleagues from all over again.  

![group](/assets/images/bsides24-group.jpg)

---

## Trainings

The main focal point for me this Bsides was a focus on training, they were too good to pass up honestly, with my main domain these days being Web and SCR.

### Attacking and Defending IIS Servers - Adrian Justice

I learnt a lot of cool tips and tricks surrounding attack vectors unique to IIS, as well as detection and mitigation strategies around them. This training can be self-guided as well if you go over to Adrian's [blog](https://zeroed.tech/blog/bsides-2024/){:target="_blank"}.   

I got to spin up and attack my own IIS server on Windows Server 2022 which is neat, never did it before :D  

- Sysadmin-ing IIS servers
    - weird visual UI bug
    - generating machine keys for viewstate
- Attacking IIS
    - Reflective loading of assemblies to never touch disk
    - File-less webshells
- Detecting and responding to all of the above
    - Memory dumps to dnspy
    - Event ID 1316 
- Evading detections of the above too
   - `Response.End` and `Response.Flush` tricks

### Efficient Vulnerability Detection with CodeQL - Josh Brown

Yet another great training. The content is too technical to summarise here, but I highly recommend doing this training or similar as the hands-on approach helps in remembering how to actually use CodeQL "in the trenches", whereas reading a blog post doesn't really translate across how "fun" to use CodeQL is. 

Setting it up is not so straightforward so having a guide is extremely valuable especially for newbies like myself. I didn't even know there was a VSCode extension for it! Makes life much easier compared to trying to debug with only CLI :relieved:  

![codeql](/assets/images/bsides24-codeql.jpg)

---  

## Badge :japanese_ogre:

This years one was simple enough for beginners to get going. It was an "orc" with a 555 timer, LEDs, diode, resistor and capacitor, resulting in a blinking badge :bulb:  

I soldered mine and found out I shorted an LED with the solder, and somehow manage to fry it while fixing it :facepalm:  Swapped out for a new one and it worked alright after that. Big thanks to `@5ud0ch0p`, `@flux` and `@0x10f2c` for letting me use their soldering setup, solder, as well as providing the 9V battery and command strips to bring the badge to life! :duck:  


> ![orcs](/assets/images/bsides24-orcs.gif)  
> Hackers making their way to the convention center each morning   

<br/>

## The talks

> 404 Not Found.  

Only attended one talk this year to show support to someone I knew, otherwise I didn't attend any others and will just watch whatever recordings come out. Seems like this year there was a number of technical issues during the main track which was unfortunate...  
 
<br/>

## K8S CTF :ship:  

I attended the Kubernetes CTF briefly and learnt a bit from the initial set of challenges, such as:
- `kubectl` commands like `get` and `describe` are tied to permission verbs like `list` and `get`
- RBAC, cluster roles
- `configmaps` and `secret` might be juicy for pentester
- pod breakouts are a thing
- as a lowly process in a pod you want to escalate your privileges to become a sigma chad "cluster admin"

![k8s](/assets/images/bsides24-k8s.png)

My knowledge in this area is quite weak, will need to revisit at some point!

## Black Bag :briefcase:

Didn't end up participating in it this time but was interesting to see they switched up the format this time around to include more online components, even including an AD component and requiring a working C2. Noticed a couple folks running around the con with a black briefcase and it was definitely humorous :joy: The final room also had lasers and roombas apprarently? 10/10  

