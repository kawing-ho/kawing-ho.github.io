---
layout: post
title: My First Android Application Test
categories: ['life', 'job']
date: 2020-07-20
---

## Foreword :thought_balloon:
I was lucky enough to test an app that did not have the following headache inducing features:
- Certificate Pinning
- Anti-Debugging checks
- Root/Jailbreak Detection
- Code Obfuscation

The phone I tested on was a <u>Pixel 2</u>, which had the following characteristics:
- A/B Partitions
- Systemless root

> I could have done it in an Android Studio emulator as well, but to get the full experience its best to use a real phone :)))  

Here's my experience and what I did!  
I took <u>3 days to setup</u> and <u>4 days testing/practicing methodology</u>...  
<div class="divider"></div>

## Setup

I first started by searching up guides on the Internet, and this blog post will most likely contain the same shit you see everywhere else <small style="font-size:0.6em">**_-- sorry in advance!_**</small>  

<br/>

### Pre-Rooting :hammer_and_wrench:


- Download the Android SDK Platform-Tools for access to `fastboot` and `adb`
- Download the Pixel2 Firmware [here](https://developers.google.com/android/images){:target="_blank"} (I used walleye 200705, use whatever)
- On the phone, open up Settings**->**About Phone**->**<u>tap on Build Number 7 times</u>
  - Now Developer Options should be enabled! :wink:  
- Go to Settings**->**System**->**(Expand Advanced)**->**Developer Options
  - enable OEM Unlock (for flashing)
  - enable USB Debugging

- The device should now show up in `adb devices`
  - You might have to press 'Allow' on a popup on your device

<br/>

### Rooting pt.1 :vibration_mode: 
- After searching through many tutorials, the only way to root the Pixel 2 is through systemless methods, and **`Magisk`** is widely used for this~  
- Simply download the `Magisk` apk file through the device PlayStore
  - Or you can [download](https://github.com/topjohnwu/Magisk/releases){:target="_blank"} it to your machine first
  - Then sideload it using `adb install magisk.apk`  

> For debugging at any time we can run **`adb shell`** to drop into the system
> Notice that at this stage we are unable to `su` just yet

- Then push the original boot image file onto the device:
  - `unzip walleye.zip; cd walleye/; ls walleye-again.zip`
  - `unzip walleye-again.zip; cd walleye-again/; ls boot.img`
  - `adb push boot.img /sdcard/Download/`
  - `Magisk Manager -> [Magisk is not installed] -> INSTALL`

- From there, use `Magisk` app to patch the original firmware file

![patched](/assets/images/mobile-select.png)![downloads](/assets/images/mobile-downloads.png)  
![alldone](/assets/images/mobile-alldone.png)

- Save the patched file to local device:  
  `adb pull /sdcard/Download/magisk_patched.img`  


<br/>


### Reflashing :zap:  

- Reboot into the bootloader using : `adb reload bootloader`
- Unlock bootloader with: `fastboot flashing unlock`
- Flash the patched boot image: `fastboot flash boot magisk_patched.img`
- Relock bootloader if you're nice :smile:: `fastboot flashing lock`
- Rebooting time: `fastboot reboot`

<br/>

### Rooting pt.2 :performing_arts:

- You actually have to go into Magisk and go to hamburger and turn on superuser access :sweat_smile:  
> ![on](/assets/images/mobile-turnon.png)  
> this is where I fucked up lol, don't forget this step! :man_facepalming:  
> After this you should be able to `su` after entering the `adb shell` :metal:

- Also a good tip is to install the **Magisk Hide** module which will repackage Magisk with a different name to evade Rooting Detection.  

- From here you can run a **_SafetyNet_** check to see if the device is rooted AND undetected, if you don't have two ticks you might be rooted but <u>detected</u>.  
  ![safetynet](/assets/images/mobile-safetynet.png)
<br/>

<br/>

### Installing stuff :package:
- objection
  - `pip3 install --user objection` (May be wise to setup virtualenv too)
- Drozer & Drozer Agent
  - The Drozer Agent to be pushed to device, and turned on within the app
    - Embedded Server: `[ON]` 
    - Port forwarding also needs to be done: `adb forward tcp:31415 tcp:31415`
    - `drozer console connect` 
- Frida
  - The server component to be pushed to device and run in the background
     ```
     adb push frida-server /data/local/tmp/
     adb shell "chmod 755 /data/local/tmp/frida-server"
     adb shell "/data/local/tmp/frida-server &" # run as root pls
     ```
  - Check by running `frida-ps -U`
- ProxyDroid
  - [Download](https://apps.evozi.com/apk-downloader/?id=org.proxydroid){:_target="_blank"} and then sideload with `adb`


<div class="divider"></div>

## Tools

### Mobile Security Framework (MobSF)  :space_invader:

- I usually install this as a Docker container for easy install + uninstall  
- ```bash
  docker run --rm -it -p 8000:8000 opensecurity/mobile-security-framework-mobsf:latest
  ```
- Drag and drop the `.apk` file and let it do static analysis
- Areas of note are: 
   - Receivers
   - Broadcasters
   - Activities
   - Strings section
   - anything interesting etc.

![mobsf](/assets/images/mobile-mobsf.png)

Useful for finding any low-hanging fruit at the start :pineapple::grapes::pear::orange::banana::watermelon::cherries::strawberry:

<br/>

### Burp Suite :ear: + ProxyDroid  :repeat_one:

Don't forget to install your BurpSuite `cacert.der` [onto the device](https://portswigger.net/support/installing-burp-suites-ca-certificate-in-an-android-device){:target="_blank"}!  

Had some trouble proxying traffic in the beginning...  
- Proxying through the Android OS Wi-Fi menu works for web browsing ---  
  <u>but not for apps:</u>  
  ![wifi](/assets/images/mobile-wifi.png)![proxydroid](/assets/images/mobile-proxydroid.png)
- **`ProxyDroid`** solves that problem as it can perform global/app-specific proxying (runs as superuser!)  
  - The following settings should be used:
     - ProxyType: **HTTP**
     - Global Proxy (or Individual Proxy/App if you're feeling confident)

<br/>

### jadx-gui :outbox_tray:
- An `.apk` is actually just a zip file containing more files in it!
- `jadx` allows us to decompile the APK and view (hopefully) the source
- Other alternatives: `dex2jar` and `jar2dex`, `apktool`
![manifest](/assets/images/mobile-manifest.png)

<br/>

### Frida + Objection :u5272::syringe:  
- Used for dynamic instrumentation, think manipulating runtime environment rather than changing code and repacking
- Objection allows for peeking into application memory, perhaps for cached data/credentials?  
  - Run with `objection --gadget "com.android.blah" explore`

![objection](/assets/images/mobile-objection.png)

<br/>

### drozer :satellite::calling:  
- Acts as a fellow malicious app on the device to see if it can intrude through exposed receivers/broadcasters

![drozer](/assets/images/mobile-drozer.png)

<br/>

### apksigner :black_nib:
- Simple check to see if the packaging/signing is done properly, making sure no important files can be tampered...  
```bash
apksigner verify --verbose app.apk
```
<div class="divider"></div>

## Methodology

I can't remember all of it but the ones that stick out the most to me are:  
- Use `adb logcat` while using application/idle to look for debugging/error info that may be useful
- Check for insecure files that have too wide cross-application permissions
- Check for Insecure Password Transmissions to the backend server
- Check for Insecure Exposed receivers/broadcaster/content providers etc
- Check for bypasses to cert pinning, anti-debugging, anti-rooting

- > and of course you have the usual web application vulnerabilities too! 

<div class="divider"></div>

## Reflections
- Apparently the **Mobile Application Hacker's Handbook** is pretty useful!
- Didn't get a chance to use it during my engagement, perhaps in the future!
   - A good standin: [**`Mobile Security Gitbook`**](https://mobile-security.gitbook.io/mobile-security-testing-guide/){:target="_blank"}
- I'm not sure if I am prepared for the monster that is **iOS testing**... :trollface:  
