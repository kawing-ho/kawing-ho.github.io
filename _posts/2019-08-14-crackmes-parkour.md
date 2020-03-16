---
layout: post
title: Crackmes - parkour
categories: ['writeup']
date: 2019-08-14
---


## Resources/Tips
- [Link](https://www.crackmes.one/crackme/5d3220cc33c5d444ad3017d0){:target="_blank"} to challenge  

<div class="divider"></div>

### tl;dr
- Disassemble the program in IDA
- **STAGE ONE**: Skip the anti-debugger by not using a debugger
- **STAGE TWO**: <u>Byte patch out a test that will always FAIL</u>
- **STAGE THREE**: Enter in the flag (plaintext in disassembly)
- _"Login"_ to the system  

<div class="divider"></div>

### Walkthrough

<br/>

#### Stage :one:  
Just by running the program initially we get this:
![fail](/assets/images/parkour-first.png)

After disassembling the program the first thing that pops up is the anti-debugger check:   
![debugger_check](/assets/images/parkour-isdebugger.png)

Which shows the following when run in a debugger:
![debugger_fail](/assets/images/parkour-debugger.png)

---

#### Stage :two:  

The program has been coded in such a way that the jump to the early exit "Please stop being false" is always taken.
![tautology](/assets/images/parkour-true.png)

Therefore, to bypass this we need to make the comparison false, and the simplest way to do it is to change either one of the zeroes into a one! 

![hexview](/assets/images/parkour-hexview.png)

By switching over to the _Hex View_ we can look at the actual machine code, and here is where we will patch the following area:  

![machine_code](/assets/images/parkour-machinecode.png)

As shown above: [`mov`](https://c9x.me/x86/html/file_module_x86_id_176.html){:target="_blank"}(C6) and [`cmp`](https://c9x.me/x86/html/file_module_x86_id_35.html){:target="_blank"}(80) instructions present along with **<u>00</u>**'s 

To patch, simply do the following:
- Put the cursor infront of the byte you want to change
- Right click and "Edit"
- Replace the value you want, in this case `01`
- Right click and "Apply changes"  
(Note that F2 can be used as shortcut too)

The binary should now be like this:  
![patched-ida](/assets/images/parkour-patch-ida.png)

Finally, we should save as a new file: _(recommend making a backup!)_
![patch-process](/assets/images/parkour-patch-process.png)

---

#### Stage :three:  
When opening the new file, we are greeted with a new message mentioning the third stage!

![parkour-final](/assets/images/parkour-final.png)

After entering the flag <small style="font-size:15px">(which can be found in plaintext in the disassembly)</small>, we successfully log in to the system :trophy:  




<div class='divider'></div>

### What I learned
- how to use IDA to patch programs (backup the original!)
- learning how to draw connections between the pseudo-code IDA generates and what the program is actually doing  

<div class='divider'></div>
