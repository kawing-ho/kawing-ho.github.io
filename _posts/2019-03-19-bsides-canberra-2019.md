---
layout: post
date: 2019-03-19
title: Bsides Canberra 2019
categories: ['life']
---

> This year I was extremely lucky to be able to sit the awesome **SecEDU** Bsides bus again! Not only that, I was sponsored by **MercuryISS** for travel, accomodation and food costs --- Thanks Edward and Alexi!

This year's venue at the **National Convention Centre** was by far the best, being located near the CBD meant there were tons of accomodation options nearby --- I only had to walk 10 minutes to get to the event compared to the 10-15 mins bus ride last year.  

<br/>

## Tamper Evidence and Locksport

During last year's Bsides I totally skipped this section, so I dedicated some hours to it this time round and wasn't disappointed. 

<br/>
I learnt how to bypass simple <u>tamper-evident stickers</u> using alcohol-based substances such as **shellite**. The theory here is that most substances have a solvent, and when you apply the solvent (using a syringe for precision) the sticker will give way, and you can reapply it afterwards

|||
|Similarly the [<u>plastic seals</u>](https://www.tamperevident.com.au/plastic-light-break-seal/){:target="_blank"} can be bypassed by simply cutting a straw, folding it into a V-shape, putting it in the hole and sliding the shim and the plastic backwards.|![sneak](/assets/images/bsides19-sneak100.jpg) |

<br/>  
The <u>wristbands</u> operate in a similar fashion to the stickers but with the added difficulty of being ribbed and also only being able to use one hand (if performing on yourself) :punch:  

![Tamper](/assets/images/bsides19-tamper.jpg)  

<br/>
While we were playing around, a physical security enthusiast from RMIT came by and gave more in-depth knowledge on the techniques as well as attempting to bypass some of the harder stuff himself. The [<u>metallic seals</u>](https://nationalband.com/flat-head-seal/){:target="_blank"} looked extremely hard to break! I believe he recommended us to go to `OzSecCon`, which unfortunately is in Melbourne, not Sydney...

<br/>

## Talks, talks, talks 
![Conference Hall](/assets/images/bsides19-ch.jpg)

I listened to mostly talks this time around, slightly controversial subject but I actually preferred single-stream talks compared to the previous years' multi-stream in different rooms. This meant that I wouldn't have to choose between two good talks happening at the same time :twisted_rightwards_arrows:  

<br/>

#### Main Track

**<u>Here are the talks that stood out to me</u>:**   
- **Apathy and Arsenic: a Victorian Era lesson on fighting the surveillance state** [[talk]](https://www.youtube.com/watch?v=egi8Lm5W3FY){:target="_blank"} [[notes]](/assets/notes/fighting_surveillance_state){:target="_blank"}

    - Would recommend any security/privacy conscious individual to watch this
    - It touches on many concepts and has great analogies
    - I felt like I was listening to a **<u>TEDTalk</u>** 

<br/>
- **Attacking JWT** [[notes]](/assets/notes/attack_jwt)
    - I learnt lots of new techniques from this talk, mainly:
        - When reencoding a tampered JWT remember to use <u>url-safe</u> base64!
        - Spoofing the signature in the event that no strict checking is done
        - Setting the algorithm field to _None_
        - Spoofing algo field to be `RSA` but actually sending `HMAC`-signed JWT
        - SQLi in a special `kid` field
        - `jku` header may lead to SSRF
    - Problem ? --- [paseto](https://github.com/paragonie/pasento){:target="_blank"}

<br/>

- **IoT Pentesting** [[notes]](/assets/notes/iot_pentest)
  - IoT making its ways into enterprise, more <u>@internetofshit</u> content!

  - Hardware presents whole new attack surface for pentesters:
      - Dump firmware
      - Get source
      - Exploit Web Interface and get RCE with vulns
  - Attack surfaces:  
      - Manual/ Data sheets from the interwebz
      - USB-oriented attacks
      - Use wifi to hook to access point, monitor traffic to cloud

  - Watch more hardware hacking videos to learn techniques such as desoldering chips


<br/>

- **Deobfuscating malicious JavaScript** [[notes]](/assets/notes/deobfus_malicious_js)
  - _<u>Compiler Theory</u>_: deobfuscation process and decompiling process similar
  - Think of deofbuscation as "optimization"
  - There are beautifiers out there but they don't preserve semantics ie:
    ```javascript
    // int convert to string
    "price = $" + 10  <=>  "price = $10"
    ```
  - Look at `SAFE` and `UGLIFYJS`
  - Techniques are so similar that you can use pre-existing pieces of compilers to build your own deobfuscater  

<br/>

- **Abusing trust in public repos** [[notes]](/assets/notes/abuse_trust_public_repo)
  - Summary:  Software supply-chain is not as safe as you thought it was
  - Pseudo-solution: [`DockEnv`](https://github.com/pathtofile/dockenv){:target="_blank"} --- virtual env for Python using Docker
    - Suitable for testing, quick (one-time) solutions and development  

<br/>
<br/>

#### Hardware Hacking Village

- There was a cool [talk](https://git.io/fxawk){:target="_blank"} about hacking a USB-powered _LED_ fan involving:  

    - sniffing the USB packets  
    - reverse engineering the protocol  
    - tampering and replaying packets  
    - in the end displaying your own patterns/ascii with the lights   

<br/>
- I also sat in to listen to <u>@gnustomp</u>'s talk about controlling your BIOS _(UNSW support!)_

    - The `coreboot` project seems quite interesting, and it is definitely a good idea if you're paranoid about the manufacturer settings being vulnerable 

<br/>


## The hardware

|:--:||:--:|||
This year's conference badge traded functionality for aesthetics <br/><br/> Which I think was definitely worth it! <br/><br/> It's called the **"<u>Nopia 1337</u>"**, and its pretty fucking awesome || It has custom firmware flashed onto it that resembles old Nokia-style interfaces <br/><br/> Complete with <br/>- Tetris<br/>- Snake :snake:  <br/>- Hidden flags <br/> - Talk schedule||![nopia-1337](/assets/images/bsides19-nopia.png) |

_(Image from the Privasec website)_    |   [(Sauce for the adventurous)](https://github.com/BSidesCbr/2019badge){:target="_blank"}

<br/>
<br/>

## I actually went sightseeing this time !
On the Sunday morning myself, <u>@adamyi</u>, <u>@jk</u>, and Lew woke up early and went to check out the Parliament House building. :sunrise: 
![Parliament House](/assets/images/bsides19-ph.jpg)

We took our time getting there _(by foot)_, unfortunately we had to rush through the building to take photos and hurriedly Uber back as time was running out and we didn't want to miss the Bsides bus back!

<br/>

## Stuff I didn't do
Conversely to last year, I did not take part in any of the competitions, ie: **ASIC IR Challenge**, **Cybears CTF** and **Drone Hacking**.

<br/>

I did network a little bit more at the event this year, but not as much as I had set out to --- _imposter syndrome_ can be a dick sometimes...

> I will try harder next year at Bsides 2020 :muscle:  
_`(but maybe by then I would have had a job already... who knows?)`_
