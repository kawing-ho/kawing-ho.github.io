---
layout: post
date: 2021-07-31
catgories: ['sec']
published: true
title: Amass Notes
---

## What's this?

> This is just my brain dump so I can easily reference again when I inevitably forget later on. :smile:  

Quick Navigation:
> [intel](#intel)  
[enum](#enum)  
[viz](#viz)  
[track](#track)  
[Db](#db)  
[dns](#dns-still-wip-functionality)  
[Other tips](#other-tips)  
[Configuration File](#configuration-file)  
[Brute-Force](#brute-force)  
[Alterations](#alterations)  
[Advanced features](#advanced-features)  
[Resolvers](#resolvers)  
[Scripting Engine](#scripting-engine)  
[References](#references)  

<div class="divider"></div>

## intel

Should be performed before actual enumeration, helps you discover additional **<small style='color:teal;'>root domains</small>** which you can add to your list of domains to `enum`:

<br/>  


Automates Reverse WHOIS lookups DNS registrar info to find similar domains:
```shell
amass intel -d owasp.org -whois
```

Grab every SSL certificate from every IP address within the IP range and return the domain:
```shell
amass intel -active -cidr x.x.x.x/xx
```
Find ASN of a corporation/org:
```shell
amass intel -org "Tesla"
```
Discovering additional subdomains/root domains via ASN:
```shell
amass intel -active -asn 394161
```

**_Recursive/Chained:_**
```shell
amass intel -asn 394161 -whois -d tesla.com
```

---

## enum

> **Three** main modes:
- passive
- normal
- active

#### passive (-passive flag)

It runs passively and collects subdomains just like most other DNS "enumeration" tools
_(The bare minimum)_
```shell
amass enum -src -ip -passive -d owasp.org
```
- Quieter and faster
- Misses out on **all** the following:
  - cyclic process
  - DNS resolutions (resolving the found subdomains for validity)
  - subdomain bruteforcing
  - permutation/alterations generation
  - recursion (doesn't try to dig deeper)


#### normal _(in-between passive and active brute-forcing)_:
- Will do reverse DNS sweeps on IP addresses around the IPs already found
- Does simple permutation
- If you turn on brute-force, it will do subdomain bruteforcing too

```shell
amass enum -src -ip -d owasp.org
```

#### active (-active flag)

Reaches out to target directly to do:
- TLS certs
- Zone transfers

- If you turn on brute-force, it will do subdomain bruteforcing too

```shell
amass enum -src -ip -active -d owasp.org
```

This mode draws a lot more attention from your target! Beware!


#### additional tips

Combining previous intel data with enum to get more results:
```shell
amass enum -d example.com -active -cidr 1.2.3.4/24,4.3.2.1/24 -asn 12345
```

Hashcat style wordlist mask for example if subdomains were found to follow a convention with "zzz-" prefixes:
```shell
amass enum -d owasp.org -norecursive -noalts -wm "zzz-?l?l?l" -dir owasp
```
---

## viz

Helps if you have a lot of data, visualization might show patterns that you have missed etc.
Formats:
- **D3** good for small targets (500-1000 nodes) (loads in web browser)
- **Gephi** or **Maltego** (`-gexf`/`-maltego`) (for 1000+ nodes)
- **Graphistry** (not sure how much to use for free)

---

## track

Run `enum` subcommand at least twice in different points in time:
```shell
amass enum -d example.com 
```
Show changes between last two amass executions:

```shell
amass track -d example.com -last 2 
```

---

## Db

Show the database data: running this when done can give more information, than simply relying on the output of one  enum command:
```shell
amass db -show -d previous-scan.com
```
Show only plaintext/raw data:
```shell
amass db -names -d previous-scan.com
```
---

## dns [still WIP functionality]
A way to take a bunch of names and provide names to DNS subcommand it will resolve the names for you.
Similar to dnsprobe, and then add them to the database, quick way to insert these names into the database, cool.
 - Not meant as replacement for DNS bruteforce
 - Used to **<small style="color:red;"><u>feed other tools data into amass</u></small>**
 - Using amass as the source of truth, ie. subdomain results aggregation

---

## Other tips
- `-include-unresolvable`, this might help with seeding permutations etc  

- amass resolves `CNAMES`, so it might not be what you want in some cases  

- IP for the Reverse DNS does not show the IP that mapped to that DNS
  - It shows for the forward DNS resolution instead, might cause mismatches
- use `-d` flag to segregate target information
  - ie. Target 1 has _target1/_ directory, Target2 has _target2/_ directory
  - Helps keep the size down too, you can delete graph database once you move on from a target, or archive it etc
- Proper way to make amass stop is to use a timeout: `-timeout n (minutes)`

---

## Configuration File

#### Blacklisting Sources
You can remove a source if you think it: 
- takes too long
- produces similar results

This can save you time, especially for larger targets

#### One Config File per Use-Case
- Make one config for coverage (slower, more results)
- Make one config for speed (faster, less coverage)
  - can be for LOTS of domains/domain-with-lots-of-subdomains etc

#### Custom Data Source
- If you make your own scripts, you can add your own sections for "Data sources", providing your own auth/key

---

## Brute-Force
- Not on by default, optional with `-brute` flag
- Slows down the process and uses a lot of memory (name generated etc) :warning:
- Might be better to do brute-forcing outside of amass, and feed it in later on

#### Recursion Tips
- `-min-for-recursive 1`, it will brute force on root domain no matter what
- Needs `n` number of names to show up so before recursion kicks in:

```javascript
amass.owasp.org -> (no other labels found so no recursion conducted)

##########################
# new run -> with 1 label!

amass.owasp.org
www.amass.owasp.org  # (min. 1 achieved)
# it now conducts recursive brute-force on amass.owasp.org)
```
- You can set it to `-min-for-recursive 0`, not recommended, you will end up waiting a lot longer
- This should be adjusted based on the target, how do they structure their subdomains for example


---

## Alterations
- On by default, can be turned off with `-noalts`

---

## Advanced features
- `-passive -include x`:
  - Useful for singling out one data source to debug for errors etc
- `-include crtsh,radb,shadowserver`:
  - Old requirements for using only one data source

---

## Resolvers

#### Amass Resolvers
- By default amass uses its own 8-10 pretty high performance DNS resolvers
- Its not the same as the DNS resolver specified within the OS
- Your network may block the use of non-compliant DNS resolvers
- Hence why some resolving works on OS but fails in amass

#### Dnsvalidator
- <u>Dont use the default resolvers</u>: 
  - if you're gonna be doing a LOT of bruteforce/alterations
- Instead you can use the `dnsvalidator` ones
- The regular ones can just be for sanity-checking/small targets etc

#### Resolover Count Importance
- Take resolvers file and take about **50** of them to `fresh.txt`
- You don't need thousands of resolvers, maybe **100-200** is the upper limit
- Some of these resolvers will give false information
- Only takes 1 bad egg to throw off enumeration, its also cyclic process
- You'll end up with `www-game.translate.dev.adminfish.target.com` 
- Cut it down to **25-50** such that the ones selected are good/legitimate ones
  - `-max-dns-queries 20000` for **50** resolvers works pretty well

---

## Scripting Engine
Scripts should live in `$(config-dir)/scripts` in `.ads` format  
- You can pull whats in your own database into the amass enumerations
- Pull data out of a file, to feed into your amass DB
- Every script needs to have:
- `name` and `type`  _(it should be one of the types in documentation)_

#### Data Source
- Your own brute-forcing algorithms etc
- Your own name-permutation algorithms
- Fire off a local program
- Accessing your own database/file

Vertical is called when amass says:  
_"okay I got domain name, I want subdomain names"_

#### TODO (Sources)
Script these passive sources as additional sources perhaps
- Dnsrecon
- The Harvester
- Vhost brute per IP? - not sure if this is possible
- https://github.com/chris408/ct-exposer
- Evilsocket/xray
- `ldns-walk`? https://dnscurve.org/nsec3walker.html  (Needs more research here)
- https://community.riskiq.com/

---

## References

[TrustedSec enumeration](https://www.trustedsec.com/blog/upgrade-your-workflow-part-1-building-osint-checklists/){:target="_blank"}

[Amass Github Tutorial Page](https://github.com/OWASP/Amass/blob/master/doc/tutorial.md){:target="_blank"}

[HakLuke Guide (intel section)](https://hakluke.medium.com/haklukes-guide-to-amass-how-to-use-amass-more-effectively-for-bug-bounties-7c37570b83f7){:target="_blank"}

[Finding other tools to add to amass scripting engine](https://www.sjoerdlangkemper.nl/2018/06/20/discovering-subdomains/){:target="_blank"}

[NahamCon (for the scripting engine part)](https://www.youtube.com/watch?v=H1wdBgY1rtg){:target="_blank"}
(1:16:34 for test script)

[RED Team Village Training Video](https://www.youtube.com/watch?v=0ZSsiH2-AwA){:target="_blank"}

---

###### Questions
Don't take concerns/troubleshooting to Github Issues.  
Better to have questions in Discord server.
