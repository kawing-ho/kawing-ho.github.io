---
layout: post
title: My First Wireless Testing Shadow
categories: ['job']
date: 2020-10-22
---

> ### Disclaimer
> This isn't meant to be a full methodology, just a record of my experiences that day!  :signal_strength:  

# Background

So I got assigned to shadow another of my seniors _(who shall not be named)_:wine_glass:  

The WiFi networks in scope were the:
- Guest wifi
- Corporate Wifi


# Set up

1) We used the ALFA-AWUS036ACH wifi adapter:
- I actually have a TP-Link lying around at home that I can use, I will try it next time in my own time
>   ![Alfa-AWUS036ACH](/assets/images/wifi-alfa.jpg)
    ![TP-Link-AC6400](/assets/images/wifi-tplink.jpg)


2) The next step was to ensure the USB is mounted to a Kali Linux VM:
- Running `lsusb` should show the adapter in the list of USB devices

<br/>  


3) From there, we ran into issues with installing the necessary drivers:  
- The package installation `apt install realtek-rtl88xxau-dkms` did nothing. 
- This is why you shouldn't always trust those random magazine-style guides on the internet :man_facepalming:

<br/>  

4) The way we got it to work was to follow this  [guide](https://github.com/aircrack-ng/rtl8812au){:target="_blank"}:   
- Note the **Notes** and **Installation** section!  
  
<br/>  

5) Finally, we check that the adapter is working with `sudo iwconfig`

<br/>  

6) From there its as easy as running `hostapd-wpe`
- in the config file you change the SSID to match that of the target ESSID

<br/>  

### Wait how does it work?

The closer you are to the rogue base station the more likely you will auto-connect to it based on your clients' familiarity with the known eSSID.

### If its such a risk then why save eSSID? 

Becase when you move throughout the building with multiple base stations, they act as repeaters of the network so you are always connected!  
<small style="color:gray;font-size:0.7em;">In theory at least. until one tries to fuck you over :)</small>

--- 

# Recon

The tool used here is `airmon`;
- to monitor the network for probes and nearby base stations
   - The probes can reveal where someone has been ie:
      - Your mobile phone will be searching for previously connecteed Free WiFi such as MacDonalds :fries: etc...  
- These base stations are ordered by signal strength
   - the lower the negative number <small style="color:gray; font-size:0.7em;">(closer to the top)</small>, the closer it is to us

![airmon](/assets/images/airmon.png)

---  

# Rogue AP / Evil Twin

The corporate WiFi uses untrusted/unverified certificates:
- This works in our favour as users are more likely to accept the fake certificates presented by the rogue AP

![wifi-cert](/assets/images/wifi-cert.png)

We use [`apd_launchpad`](https://github.com/WJDigby/apd_launchpad){:target="_blank"} to:
- generate a fake certificate
- stick the fake certificate and fake eSSID into a `hostapd-wpe` config file

Then we re-launch `hostapd-wpe` which will display a visually similar error and certificate name


### Now we wait

At this point it is a waiting man's game.  
We have to wait for users devices to send auto-cached credentials...  
  
However in this case because of the existence of **certificates**, user interaction is :100:% required.  
> <u>They have to click Accept before the authentication prompt pops up!</u>  

---

# Deauth Attacks

We decided to use a more active approach when still no results came in.  
The `aireplay-ng` tool can be used to send crafted beacon frames to  chosen APs in the vicinity.

The idea is to send deauth packets to boot users off the legitimate base stations, and catch some of those with our rogue AP who are attempting to reauthenticate with the legitimate base station.  

---

# Result
The overall outcome of the first day of testing was not so good, as only <small style="color:brown;">5-6</small> people were in the office at the time---  

Here is what a captured authentication response would look like:  
(When someone enters their username and password into the prompt)  
![wifi-hash](/assets/images/wifi-hash.png)

The hashes can then be sent to `asleap` or `hashcat` to be cracked offline!  

---

# Further attacks

Of course how far this attack can go in terms of depths is determined by how secure/insecure the client network setup is like:
- Do they properly segregate the interal corporate WiFi with the Guest WiFi?
- Do users reuse passwords across both networks? Or their RDP/VPN logins?
- Are there any services running on the internal corporate network that can lead to a foothold scenario? ie. mySQL / FTP / webservers...  
