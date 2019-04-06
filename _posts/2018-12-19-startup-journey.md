---
layout: post
title: Summer internship at a startup
date: 2018-12-09
categories: ['operations', 'job']
---

> I worked as a casual intern over summer at a lovely place called **STEMN**!   
  I worked under <u>Jackson Delahunt</u> with two other interns from UNSW as well  

<div class="divider"></div>
### How was it like?
This was my first time working at a **startup**,  
therefore I got to experience the pros and cons that came with it ------  
<br/>

:heavy_check_mark:  
- Wearing whatever you want  
- Working on literally the couch or kitchen table or the floor if you want  

- Super flexible working hours and/or presence in office  
  (I usually avoid remote work though as I get easily distracted at home)

- Home office has an awesome rooftop and two cats :cat:  
  (unfortunately terrible if you have allergies)  

- Occasional treats such as tea or ice creams to boost morale :tea:

- Very flat hierarchy  
  (can share memes around and sometimes watch educational videos on the TV)   
<br/>

:x:  
- The payroll was usually late :money_with_wings:   
  (luckily I had a cushion to accomodate for delays)


- No solid goals or deadlines  
  (can be a good or bad thing depending on how you look at it I suppose _???_)

<div class="divider"></div>

### What were my contributions?  

> There weren't really a fixed set of requirements for my role, more of you just handle whatever is thrown at you, and I was told I did that pretty well :wink:   

<br/>

#### Sec
In the beginning I had a chance to apply what I had learnt in **`COMP6443`** by performing a week-long penetration test of the `dev` STEMN web application   

I tried most of the **OWASP Top Ten Vulnerabilities** ranging from `XSS` to `SQLi` but found nothing exploitable in that timespan  

As part of my _**<u>Chaos Monkeying</u>**_ role I also fuzzed API endpoints and discovered faulty logic that would crash the backend when certain requests were made  
  
During a white-box test (I was given credentials to the remote GCP servers) I found unsecured cloud-meatdata directories and alerted Jackson to the risks involved and proposed a fix

<br/>

#### Dev/Ops

Created a packer image, also shared some knowledge with the founder such as asciinema
sepnt two weeks reversing -> and compiling yarn
Jaeger tracing + yarn

Wrote a task-runner to replace yarn as it was too slow

<br/>

#### Misc
- Became a guinea pig for new intergrations into the tech stack

did some tweaking with Weechat... more of a time consuming thing tbh, not much expertise required :)
- did some quick fixes to improve workflow such as pre-commit hooks (using husky) as well as hub pull requests

recently added a quick script to make splits possible (ie. select changes to commit to x branch vs y branch)

<div class="divider"></div>

### What did I learn?

- In addition to the things mentioned above, Tons of things really
- Learnt more about Node.js
- Hands on with IAC technologies such as Packer, vagrant, Terraform -> generate pre-baked images
- I liked terraform so much I made my own project to maximize my Digital Ocean credits

- Kubernetes -> hardening and attack
- it seems quite complicated, this whole notion of pods/ nodes/ contexts
- quite cool how a kubectl given a context file and communicate with a local cluster or a secured cluster in the open
- certificates and RBAC

- yarn shim task runner, some Javascript/Typescript nuances
- lifeyccle of cloud applications, stuff like continuous intergrations, hashicorp utilities such as terraforming 
git aliases are awesome

Typescipt things :)

<br/>

#### Knowledge exchange :)  
bash syntax such as !$ <-- used to get args of previous command, rtv (reedit terminal viewer haha)
taught about various CLI quirks too -> apropos 


he keeps me up-to-date with bleeding edge tech such as istio and others 

He containerized his entire desktop application, so it is accesible from any compter.

Not only that he setup his home desktop to become available from any web browser. Right now he is relying on securoty through obscurity but I will advise him to change that hahaha


<div class="divider"></div>

### April 2019 Update

During summer I used to come in almost every weekday; Nowadays to balance with my other jobs at UNSW I only come in 2 days a week --- The workload has slowed down significantly as well...

<br/>
Since the very beginning of my internship I told Jackson I wouldn't stay for long as I wanted to progress down the security route and this job wouldn't exactly do it for me, so I am now parting ways with **STEMN**, all the best to them in the future! 


> Overall really awesome learning experience, I learnt not just technical knowledge but also heaps of life stuff as well ~   
(such as spiritualism, productivity etc...) :tanabata_tree:   


