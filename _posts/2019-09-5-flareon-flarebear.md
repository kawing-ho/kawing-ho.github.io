---
layout: post
title: FlareOn'19 - Flare Bear
categories: ['writeup']
date: 2019-09-05
---

## Resouces/Tips
Tools I used:
- `jadx` for decompiling and viewing the files (`dex2jar/jar2dex` works too!)
- `Anbox Application Manager` to run the app
- `adb` for sideloading into the emulator and debugging  
- `VisualStudioCode` for looking at the files :astonished:  



> We at Flare have created our own Tamagotchi pet, the flarebear. He is very fussy. Keep him alive and happy and he will give you the flag.

![1](/assets/images/flarebear-1.png)
<div class="divider"></div>

## tl;dr
- Do lots of reversing...   
- ~~Fuck up a lot of times~~ Backtrack and stop overthinking the challenge

- Basically you need a proper combination of three actions to trigger the flag state in the bear  

<div class="divider"></div>

## Walkthrough

Lets see whats inside the provided `flarebear.apk` file:  
```shell
$ jadx -d flarebear flarebear.apk
 ...
$ ls flarebear
 resources  sources  
```

![dirlist](/assets/images/flarebear-listing.png)

Using `anbox` and `adb`, we install the game into an emulator to see what it is all about!  
```shell
$ adb devices
List of devices attached
emulator-5558	device
  
$ adb install flarebear.apk
  ...

```

After installing through `adb`, we can see the application within `anbox`! 
![anboxing](/assets/images/flarebear-anbox.png)

We can then interact with flarebear and see what game mechanics are present:

![game_mechanics.gif](/assets/images/flarebear-game.gif)

You can:
- Load Game/ Create New Game
- Give your bear a name
- Feed it
- Play with it
- Clean it

Understandably the bear starts out happy but if you do not feed/play and clean too much it will become sad, feed too much and there will be poo around the screen !!!

This coincides with this function in one of the UI class files: 
![ui-run](/assets/images/flarebear-uirun.png)


Looking at the **`setMood()`** function, we see some interesting things:  
![setMood](/assets/images/flarebear-setmood.png)

Perhaps the most notable one is <small style="color:red;">`danceWithFlag()`</small> !  The function can only be reached if two conditions are met --- `isHappy()` and `isEcstatic()`  


![flarebear-happy](/assets/images/flarebear-happy-ecstatic.png)

As shown above the conditions for `isHappy()` ---  f/p must be between 2 and 2.5

For `isEcstatic()` it is shown in teal...  

Going back to <small style="color:red;">`danceWithFlag()`</small>:  
![danceflag](/assets/images/flarebear-danceflag.png)

Another two functions of interest are `getPassword()` and `decrypt()` :closed_lock_with_key:

![getpassword](/assets/images/flarebear-getpassword.png)
![getpassword2](/assets/images/flarebear-getpassword2.png)]
![decrypt](/assets/images/flarebear-decrypt.png)

They seem quite complex so we will leave them for later...   

From previous screenshots we see that there are `getStat()` and `getState()` functions, perhaps there are `setStat()` and `setState()` functions too!  

![getstat](/assets/images/flarebear-getstat.png)

Based on what we've seen so far, `getStat()` can take inputs of `f`,`p`,`c` and will return how many times that character occurs in a global Activity string    

At this point you might have guessed what those characters stand for:
- **f** for food :plate_with_cutlery:
- **p** for play :basketball:
- **c** for clean :shower:
 
![getsetstate](/assets/images/flarebear-getsetstate.png)

It seems like the **State** is being stored in some kind of in-memory registry/global dictionary. Get/Set would retrieve/edit the values based on the key given.  

There was no `setStat()` function, however `saveActivity()` comes pretty close in terms of functionality...  
![saveActivity](/assets/images/flarebear-saveactivity.png)

After this, we can move on to the three functions which control the Activity string as well as the three States  

![triple](/assets/images/flarebear-triple.png)

We now know that the only way to reach the `isEcstatic()` state is to have a right combination of plays, cleans and feeds _(in no particular order)_ which satisfies the two requirements above!

Since it becomes an **Algebra** problem <small style="font-size:0.8em;">(solving for four variables over three equations)</small>, I wrote a script to help in this process:

```python
#!/usr/bin/python3
# For spamming flarebear stats :)
# Winning combo: ffffffffccpppp

mass = 0
clean = 0
happy = 0

activity = ""
choices = ['f','p','c']

while True:
   
   choice = input("Next move? (f/p/c): ").lstrip().rstrip()
   if(choice == 'q'): exit()
   if(choice not in choices): continue
   activity += choice

   if(choice == 'f'):
      mass += 10
      happy += 2
      clean -= 1

   elif(choice == 'p'):
      mass -= 2
      happy += 4
      clean -= 1

   elif(choice == 'c'):
      mass += 0
      happy -= 1
      clean += 6

   print(f"\nMass: {mass}\t Clean: {clean}\t Happy: {happy}")
   print(f"Activity: {activity}")

   # winning combo   
   if(mass == 72 and happy == 30 and clean == 0):
      print(f"[WIN] ::> {activity}")
      break

#=============================================================

```
>  An important thing to note is that too many 'f's can cause the other stats to spiral out of control, this is preluded by the 2.0 and 2.5 range checks we encountered previously. _[KISS principle applies]_

<br/>  

  
Thus I employed this strategy:
- 8 x `f` to get 80 __mass__ --- However, this causes __clean__ to be -8
- So we do 2 x `c` and bring __clean__ up to positive levels
- We then finish off with 4 x `p` --- too many __clean__ and not __happy__ (only 14)

With this final combination, we get the flag from our happy bear :bear::checkered_flag:  
![win](/assets/images/flarebear-win.gif)

<div class="divider"></div>


## What I learned  

- First proper foray into Mobile Reversing... sort of.
- `smali` is almost like a intermediate language between `dex` and source, similar to assembly  
- Somtimes its better to follow the natural order of reversing than to try and reverse from back to front...  
   - I spent too long looking at the `decrypt` and `getPassword` functions
   - I ended up neglecting the actual challenge involving State and Stats  

