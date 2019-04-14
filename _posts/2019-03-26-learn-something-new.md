---
layout: post
date: 2019-03-26
title: Learn something new everyday !
categories: ['sec']
---

> I was given the <del>privilege ?</del> task of recording COMP6[48]43 lectures.
Usually the content is repeated (since I took the course last year)   
But this time Norman actually mentioned something that is quite useful in real-world scenarios !

# Technique explained

Some sites out there that still run `ASP.NET` may have this thing called **`trace.axd`** which <u>logs requests</u> made to their site.  
- usually should be turned off in production
- mainly used for debugging and sysadmin-ing

||
|Obviously I dont have to tell **<u>YOU</u>** the reader to be ethical and to use this technique at your own disgression|

<br/>

# Why its useful

> Think **`RequestBin`** but its actually someone else's server!  

Usually when you want to exfiltrate data/steal cookies over HTTP you would:
- set up a listener
- exfil the data as a query param in the URL

This is fine and dandy but what if the victim logs your IP and decides to trace you? :scream: 

___

<br/>

This technique although not foolproof, does provide one step of indirection for an attacker to exfiltrate data.
The victim would then have to contact the server owner (of the site running `trace.axd`) for the IP.  :squirrel:

<br/>

#### Extra indirection you say?  
You can chain this for more confusion:
- using ephermeral web browsers such as [`Browserling`](https://www.browserling.com){:target="_blank"}
  - From there you can copy and paste the data into PasteBin from the remote browser, get it locally and set the paste to expire

- using the page's complimentary `[clear current trace]` button... 
  - This allows you to clear the screen after you are done!
  - They might still store logs in the backend though so watch out 

- do your `cURL`-ing from an AWS box or similar VPS  
  - it is double-edged though because if AWS gets contacted to reveal you I'm pretty sure they will  
  - However if you have some way to anonnymously access someone else's remote server then it will work!
    - <del>with the side-effect of the blame being shifted onto them of course</del>  

- Cherry on the cake: Turn on your trusted **VPN** service while doing this :cake:  

<br/>

# Practicalness // Demo time


#### How to find one such site? 
Google is your friend,   
with the right _Google Dorking_ we can look for sites that match :mag:  

- we can search for the string **`inurl:"trace.axd?id"`**
- this basically means --- find all URLs with `trace.axd` in it! 
  - the `id` part is just to get rid of white noise such as articles

![google-dorked](/assets/images/learn-1.png)

In this demo we select this site to use: [http://www.frikirke.dk/trace.axd](http://www.frikirke.dk/trace.axd){:target="_blank"}  

<br/>

#### And then ?

We might have to clear the screen if it is full (in my case it was). A new trace request will not be shown if the existing buffer is already full.

![trace-full](/assets/images/learn-2.png)  
<br/>

#### Tada!

Using an AWS box I own, I send a request to this site with my fake payload:  
![aws-curl](/assets/images/learn-3.png)

And we see the result here!  

![trace-result](/assets/images/learn-4.png)

![exfil-complete](/assets/images/learn-5.png)

From there we can clear the trace page away (if we want to)  

<br/>

From the victim's prespective: 
- they only know that a request was made to this random webpage
- not a suspicious URL like `evil-listener.pwn` :passport_control:  
