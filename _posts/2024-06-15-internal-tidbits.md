---
layout: post
date: 2024-06-15
categories: ['work', 'sec']
published: true
title: Internal Memory Dump
---

# Background

Just wrapped up an Active Directory Pentest and learned quite a lot of things! Some of this I figured out myself, some with the help from the talented `@slygoo` :smile:  

Gonna do a write-up/dump so I can refer back in the future...  

---
   
# Tips & Tricks

- Powershell `$FormatEnumerationLimit=-1` will make it so your output doesn't get truncated if you output it in Table / List format.  


## Random Pentest Tool knowledge

- `cme`/`netexec` has an audit mode which auto-masks all passwords. You can set it in the config file
-  similarly you can also change the `Pwn3d!` text to something more formal if you wish

- `Group3r` is amazing at displaying Group Policy at a glance, and also helps you find "findings"  
- Always try using native Windows methods over attack-tooling as this is less likely to get caught.
    - `Enter-PSSession` vs `Evil-WinRM`
    - `RDP` vs `xfreerdp` 
    - `CMMC` vs `net`
    - `ADExplorer` vs `SharpHound` etc.
  
- `go-secdump` - another alternative remote dumping tool
- `DPAT` module in `Max` - great domain password audit tool that also shows breakdown by domain groups/categories etc.   


## Random AD knowledge

- `SeMachinePrivilege` controls who / which group can add computers to the domain. One hardening step is to set it to Administrators only
- GPOs are a great way to catch overprivileged assignment of privs against a group of principals for example  
- SPNs don't have to be valid, so if you see a user with an invalid but non-empty SPN string call it out as it only increases the risk of compromise as that user can now be kerberoasted.  


## LDAP / Relaying

- LDAP Signing is for protecting LDAP 389
   - You cannot relay SMB to LDAP 
   - UNLESS you combine with NTLMv1 and Drop the Mic
   - You can relay HTTP to LDAP, either through link clicks or by using WebDAV + Coercion etc. Syntax is `\SERVER@PORT\PATH`
- LDAP Binding is for protecting LDAPS 636
   - You can technically auth SMB to LDAPS but not relay 
- You can [smuggle sensitive LDAPS operations through LDAP](https://offsec.almond.consulting/bypassing-ldap-channel-binding-with-starttls.html){:target="_blank"} if signing is OFF but binding is ON via StartTLS, modern versions of impacket handle this transparently already :) 

## MSSQL

- Don't discount MSSQL servers / service accounts as weakly configured permissions may let you escalate on servers or hop around links  
- When `PowerUpSQL` says a server is 'Accessible', it has filtered down to servers that are a) connectable and b) your user has privs
  - Therefore, there might still be a couple servers out there that are browsable, just you might need a different user :)   
  
## RBCD

- Is not just about adding users, its a two part attack
  - You can add computer or trick a privileged entity to add a computer account you can control
  - Either a target machine or service is misconfigured, OR you have write permissions over that object
  - Can add the DelegateTo attribute on target and set it to our computer object
  - Our computer object can then delegate as a user against target service similar to silver ticket once granted privs

## Navigating BloodHound 

- Inbound Object Controller - what things can control current account
- Oubound Object Controller - what things can current account control
- In some views, nodes can be further 'Expanded' to reveal all sub-nodes, handy for viz  
- Direct Membership / Unrolled Membership - see what groups current account belongs to
- Direct Members / Unrolled Members - see members / sub-members of a group
- `space` can be used to see all nodes on a graph
- `ctrl` can be used to toggle node labels on/off
- start and end nodes can be used to plot a nice graph showcasing an attack path
