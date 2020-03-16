---
layout: post
title: FlareOn'19 - Memecat Battlestation
categories: ['writeup']
date: 2019-08-17
---

## Resources/Tips

- [Link](https://flare-on.com/files/Flare-On6_Challenges.zip) to all the Flareon'19 challenges  :zap:  
- Use **`DNSpy`** for this one --- It's good for disassembling **C#** to _almost_ source!  
- Know how **XOR** encryption  works!

<div class="divider"></div>

### tl;dr
- A double-stage challenge:

- **STAGE ONE**: PLAINTEXT IN SOURCE!  
- **STAGE TWO**: XOR DECRYPTION!  

![intro](/assets/images/meme-intro.png)  

<div class="divider"></div>

### Walkthrough

Apparently C# decompiles to source in **.NET**, who would've thought?
![dnspy](/assets/images/meme-decompile.png)

Looking at the main function we see the following:
![main](/assets/images/meme-main.png)

There are two stages to pass which then brings us to victory! Also the `WeaponCode` properties seem interesting, which warrants closer inspection...    
  
---

Stage :one:  

![stage-one](/assets/images/meme-stage1.png)

As we see from the screenshot above, entering the wrong code gives an <small style="color:red;">"Invalid Weapon Code"</small> message, lets see where this is invoked!

![invalid](/assets/images/meme-invalid.png)

The string leads us to the `stage1Form` from earlier, and we discover the following function which referrences it:  

![rainbow](/assets/images/meme-rainbow.png)

From here the plaintext string **"RAINBOW"** is identified to be the code for stage1 ! We enter it and watch the magic happen: :rainbow:  


![rainbowgif](/assets/images/meme-rainbow.gif)

---

Stage :two:  

We know all that jazz so we skip straight to the **`FireButton_Click`** function in the `stage2Form`.

![stage2](/assets/images/meme-stage2.png)

This time we see that the input string is being compared in a new function:   
![isvalid?](/assets/images/meme-valid.png)

The function returns True if the input string XOR'ed with 'A' matches that random obfuscated string there...  

After writing a simple Python script to solve, we get the code~  
```python
#!/usr/bin/python

obfus = ['\u0003',' ','&','$','-','\u001e','\u0002',' ','/','/','.','/']
original = []


for char in obfus:
   
   dec = ord(char)

   # Direct XOR'ing between characters not doable in Python
   new_dec = dec ^ ord('A')

   new_char = chr(new_dec)
   
   original.append(new_char)

print("".join(original)) # ~~~~~~~~~~~~~~~~~> Bagel_Cannon

```

After entering the code and watching the cool animation we get the flag!  
![victory](/assets/images/meme-victory.png)

<div class="divider"></div>
### What I learned
- How to use `DNSpy`
- The rough layout of **C#** programs and how they are packaged
