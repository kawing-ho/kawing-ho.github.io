---
layout: post
date: 2020-12-25
title: First bug accepted ever
categories: ['life']
---

# Background
I was extremely lucky and privileged to be referred to join the platform:  
{:refdef: style="text-align: center;"}
![synack](/assets/images/synack.png)
{: refdef}

This is where I will be doing most of my bountying for the forseeable future, it has its perks but also some of its frustrations I will say...  

# The bug

The functionality in question is one of those <u>"Send to a friend"</u> things you see on most eCommerce websites. 

It essentially allows you to specify an email address and it will share the current item/article you are browsing to said friend. 

In terms of the raw HTTP POST request, there were the following parameters:
- **from**
- **to**
- **subject**
- <small style="color:red;">**body**</small>

The outgoing request looked like this:
![request](/assets/images/bug-request.png)
And the corresponding email:
![email](/assets/images/bug-email.png)

After numerous attempts at tampering with the other parameters, and even the message itself, I found that the message would always revalidate back to the old formats.  

However, there seemed to be a separate validation occuring for the **URL** within the body, as an illegal URL (not the same as the hostname) would result in the following email:  

![null](/assets/images/bug-null.png)

> I then remembered the great talk given by Orange Tsai on  <br/>
> [bypassing URL parsers](https://gsec.hitb.org/materials/sg2017/D1%20-%20Orange%20Tsai%20-%20A%20New%20Era%20of%20SSRF%20%e2%80%93%20Exploiting%20URL%20Parsers%20in%20Trending%20Programming%20Languages.pdf){:target="_blank"}.  :orange:   (see slides 28-30)

The one method I tried straight away which worked was inserting the following URL:
```ruby
https://good-allowed-url@evil.com/
```

This fools the parser as it checks if the good hostname is present in the URL string, however the browser would interpret it as `https://evil.com/`

![evil](/assets/images/bug-evil.png)

An attacker could therefore use this to phish unsuspecting users, if they do not inspect the URL properly...  :fishing_pole_and_fish: :email:


# Payout...?
In the end I was paid out **$118.97** USD for this vulnerability.

For an additional parameter on a duplicate endpoint it is worth 10% of the original price (which I guess was $187), but the additional $100 came from the program being in Blitz :zap: mode.  

> Blitz mode is when a program offers more bonuses to researchers when a program goes stale/ no reported vulnerabilities for an extended period of time 


# Reflection

I honestly thought my submission would be marked as Duplicate as someone had already made a submission for the same endpoint.  

I was extremely grateful for an early Christmas present when the email came in saying it was accepted!!!  :gift:  

It's nothing spectacular compared to what other stories you see/hear out there, but this is my achievement and I'm proud of it nonetheless :smile:  
