---
layout: post
title: Current State of my Bug Bounty Methodology
categories: ['sec']
date: 2020-10-23
---

> ##### TL;DR
> This is just my way to compare to how shit I was back in uni, and also a referrence for anyone who asks me what my methdology is.<small style="font-size:0.5em;">(2020)</small> :wind_chime:  
>
> ---
>
> I have my seniors at HackLabs and Pure.Security to thank for the **1+** years of guidance!  

## Summary Graph :chart:

> ![INSERT GRAPH HERE](/assets/images/bug2020.png)

<br/>

## Subdomain Bruteforcing :1234:
#### `amass`
- Great for first-step recon, does both passive and active scanning  

- ```shell
amass enum -df ./domains -v -ip -o amass.txt -active -r 8.8.8.8,1.1.1.1 -w /tmp/subdomains-top1million-20000.txt -aw /tmp/words.txt
```  
---

#### `altdns`  
- Used with subdomains gathered from `amass` and permutated with a wordlist
- Example wordlists: [SecLists subdomain wordlists](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS){:target="_blank"}

#### `dnsvalidator`
- Pre-requisite step before using `zdns`
- checks to see which DNS resolvers can be used and provide accurate results
- ```shell
dnsvalidator -tL https://public-dns.info/nameservers.txt -threads 20 -o resolvers.txt
```

- Alternatively: [SecLists static list](https://github.com/danielmiessler/SecLists/blob/master/Miscellaneous/dns-resolvers.txt){:target="_blank"}

#### `zdns`  
- Blazing fast but also prone to false positives and the following errors:
   - <small style="font-family:Consolas;">TIMEOUT</small>  --- if the timeout value is too low
   - <small style="font-family:Consolas;">SERVFAIL</small> --- if the load is not distributed as wide as possible
- Requires `ulimit -S -n 1048576` otherwise will complain about not enough file descriptors 
- Useful pipe output format: `jq -r '.name + "\t" + .data.answers[0].answer'`
- ```shell
zdns A -name-servers @./resolvers.txt -result-verbosity short -retries 3 -timeout 30 -output-file <OUTFILE>
```

#### `dnsprobe`  
- Used to perform final validation on the newly discovered subdomains

---

#### `httprobe`  
- Used to quickly discover webservers on standard ports across hosts

#### `aquatone`  
- Useful for a large scope
- Looks for web content, browses to it and takes screenshots
- Sometimes inaccurate (may follow redirects)


<br/>


## Webserver/Cloud level 
#### `enumXFF` 
- bypass certain CDN-level/Origin-level WAF restrictions

#### `Burp Intruder` 
- spam request headers against the webserver  
- Example Wordlist: [SecLists request headers](https://github.com/danielmiessler/SecLists/tree/master/Discovery/DNS){:target="_blank"}


<br/>

## Directory Bruteforcing :open_file_folder:  
#### `dirsearch` 
- for one host with no WAF/rate limiting
- blazing fast with surprisingly accurate results
- careful not to get IP-banned by hosts!

#### `meg` 
- for wide scope to avoid IP-ban/rate limiting
- Cycles each wordlist entry through each host, the more hosts the safer??

<br/>

## BruteForcing through time! :telescope:  
#### `waybackurls` 
- useful when you can't find anything with DirBruteing
- chances are the paths from 2-3 years ago might still be active
- Similar tool is `gau`

<br/>

## JavaScript file parsing
#### `LinkFinder` 
- good for scraping URLs etc
- simpler version: `relative-url-extractor`
<br/>

## XSS
#### `xsshunter` 
- for Blind XSS payloads and capturing successful triggers

<br/>

## SQL Injection
#### `sqlmap` 
- prepare to get IP banned
- sometimes it refuses to work, most likely you need `--force-ssl`

<br/>

## Misc. info :8ball:
#### `gf` 
- useful for searching through loads of output for multiple things
   - ie. exposed git repo ---> use `truffle.json` for tokens/secrets/api keys

#### `Burp Active Scan` 
- sometimes when you ran out of ideas this might just give you a new lead!

#### `Sites`
[`https://transformations.jobertabma.nl/`](https://transformations.jobertabma.nl/){:target="_blank"}  
[`http://bugbountytips.me/`]( http://bugbountytips.me/){:target="_blank"}  
