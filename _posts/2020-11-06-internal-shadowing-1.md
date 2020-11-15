---
layout: post
title: My First Internal Infrastructure Shadow
categories: ['job']
date: 2020-11-06
---



## Setup Phase :construction:

#### Kali VM :dart:
- Preferably installed on a physical host machine
   - Tried installing on a VM using nested virtualization but didn't work
   - VMWare Player refused to work, but somehow the godsend that is Virtualbox worked!

- Give VM lots of RAM! The host machine had 18GB of RAM so Kali got 8GB :smile:

- Set the VM to <u>bridged mode</u>:  
   - The VM gets its own IP address instead of sharing network with host

- Got the client sysadmin to:
   - Configure static internal IP via DHCP for the Kali VM
   - SSH forwarding from WAN-facing host to internal Kali VM
   - Only allow SSH traffic from our whitelisted locations, eg. office/VPN


- Change the `root` password to something random! :moyai:  

#### Nessus Professional :bento:
- Download the package and install **over cli**
- Use license key to initialise and setup 
- run the scan and watch as the HIGH/CRITs flow in :wink:  
   - BlueKeep
   - BlueGate (no public PoC)
   - SigRED (no public PoC)

#### Metasploit / msfconsole :space_invader:
- Use the `auxiliary/scanner/ipmi_dumphashes` module
   - Used to dump the IPMI2 hash of common users on vulnerable machines

<div class="divider"></div>

## Tools :hammer_and_wrench:

#### responder :satellite:

- "Responds" to requests over the network that are meant for other servers
- Typically stuff like `wpad` files or SMB-related requests
- By doing so the requesting server gives the credentials to the "responder"
- The captured NTLMv2 hashes can then be cracked offline
- Can also be relayed using `MultiRelay`


**Note on NTLMv2:**
- Has nonce so can only be used once, great for anti-forgery
- NTLMv2 can be cracked using hashcat for the NTLM hash, or the plaintext
- NTLM hash doesn't come with a nonce so it can be replayed indefinitely

#### Multi Relay :arrows_counterclockwise: 

- Useful when you just can't crack the hash 
- Perfect for servers with **SMB Signing Disabled**, as forged hashes accepted
- Use the hash before the legitimate user can, race condition scenario??  
- Kind of like stealing someone's bank card in the bank, jumping queue infront of them, doing stuff as them and then throwing the card away... :credit_card:   

#### crackmapexec :dash:

- Spray a NTLM hash or password across an IP range
- Have to specify if domain account or local account as they are different
- Very noisy, after 5 failed attempts the account will be locked for 30 mins across whole domain
- Useful in finding valid credential sets and accounts quickly 

#### bettercap :twisted_rightwards_arrows:
- Not much chance to play with it yet 
- _Something something network swiss army knife_
- Can fuck with other attached networks 


#### Active Directory Explorer :statue_of_liberty:
- Locate IP address of any Domain Controller (DC) 
- Port forward `port 139 (LDAP)` to local machine  
- Use AD explorer with any domain account to query DC for info
   - Can perform very powerful searches
   - ie. Give me all computer/workstation names and OS version across domain
- Weakness lies in standalone nodes that aren't part of AD

#### Bloodhound :dog2:
- Used in conjunction with AD explorer to compute the fastest path to DA
- Extremely noisy, not to be used in situations requiring stealth

<div class="divider"></div>

## More to come :soon:

Of course this is not an exhaustive list of what can be done, it is simply my experience of this test. There may be more internal infrastructure blogs to come...  
