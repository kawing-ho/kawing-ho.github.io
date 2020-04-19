---
layout: post
title: FlareOn'19 - dnschess [walkthrough]
categories: ['writeup']
date: 2019-11-03
---
<br/>  

## Foreword/Disclamer

I didn't manage to solve this challenge the first time...  
After sinking in 30 hours and having enough <small style="font-size:0.7em;">(I took the completionist route for this one)</small> I decided to look at the solution. :man_facepalming:  

Now several months later, I reapproach this challenge with the intention of writing a full walkthrough, this post is gonna be a long one, so strap in  :wink:  

> Some suspicious network traffic led us to this unauthorized chess program running on an Ubuntu desktop. This appears to be the work of cyberspace computer hackers. You'll need to <u>make the right moves</u> to solve this one. Good luck!
<div class="divider"></div>

## tl;dr

- Analyse the `.pcap` file and the functions :mag:  
- Figure out the relationship between the program checks and traffic  
  **_(I got severely stuck here)_**
- Replay the right actions (as the flavortext suggests)
- Get the flag  
<div class="divider"></div>

## FULL Walkthrough

We are given three files to work with (excluding the message):
![files](/assets/images/dnschess-files.png)

Lets launch the **`ChessUI`** <small id="gif"></small> and see what happens:  
![first_play](/assets/images/dnschess-firstplay.gif)

<small style="font-size:0.7em; color: lightgray;">That's weird, I don't think the game should act this way...</small>

Thankfully there is a **`capture.pcap`** which shows a previous game session, lets open it up in `Wireshark` now:   

<div align="right" style="font-size:0.7em;">(Click to enlarge)</div> 

[![wireshark](/assets/images/dnschess-wireshark.png)](/assets/images/dnschess-wireshark.png){:target='_blank'}

Some key things to take away from this pcap file:
- It is played with **DNS requests** and **DNS responses**, hence the name `dnschess`  
- None of the requests/responses go outside of the local/internal network :thinking:  
  - The Source/Destination of packets are within the `192.168.122/24` subnet
  - While the wonky subdomains all resolve to within the `127.0.0.0/8` subnet  
    ![records](/assets/images/dnschess-records.png)
- Each movement set _<small style="font-size:0.7em;color:gray;">ie. Rook to C3-C6</small>_ will resolve to a different internal IP  
- The Nameserver used points to localhost! This further strengthes the theory that everything is run internally  

<br/>

We also have a quick peek at the **`ChessUI`** disassembly:  

The first thing that we notice is that because the ELF is stripped, 99% of the <u>original function names</u> are not present. A new aspect of the challenge is to go through each function and make assumptions about its role in the program and name it accordingly...  

![comparison](/assets/images/dnschess-funcompare.png)

For clarity I will go into detail with **`ChessAI.so`**, `ChessUI` is full of functions which are not critical to the challenge and mostly handles UI etc...  
![chessAI.so](/assets/images/dnschess-chessaiso.png)

|---|
|We see that some symbols are already given, usually this means that those functions are ready for export and used by our main program **`ChessUI`** ---    |![exports](/assets/images/dnschess-exports.png)|

`getAiName` and `getAiGreeting` are fairly straightforward functions which pretty much just return a string from memory. 
![namegreet](/assets/images/dnschess-namegreet.png)

Lets inspect each of the <small style="color:darkorange;"><u>unknown functions</u></small> above...  

<br/>  

As a preface, [<small style="font-size:0.8em;">constructors and destructors</small>](https://stackoverflow.com/a/30972459){:target="_blank"} specified in ELF headers are used as part of the <u>initialisation</u> and <u>termination</u> on runtime for programs as well as on dynamic linking for shared objects. 
![linker-graph](https://cdn-images-1.medium.com/max/741/0*oQA2MwHjhzosF8ZH.png)

#### start 
By using the <small style="color:skyblue;">xrefs</small> (cross-referrence) function, we can see below that `start` is the <small style="color:purple;">entry point into the program</small>; It is also called by <small style="color:red;">`sub_1100`</small>:  
<div align="right" style="font-size:0.7em;">(Click to enlarge)</div> 

[![xrefs](/assets/images/dnschess-startxref.png)](/assets/images/dnschess-startxref.png){:target="_blank"}

#### sub_1100
This <small style="color:red;">`sub_1100`</small> is actually called as part of the `_fini_array` of functions during destruction --- as shown under **_ELF Termination Function Table_** :  

[![sub1100](/assets/images/dnschess-sub1100.png)](/assets/images/dnschess-sub1100.png){:target="_blank"}  

#### sub_10C0 and sub_1140

`sub_10C0` is called by `sub_1140`, which in turn is called as a part of the **_ELF Initalization Function Table_**:   
[![sub10C0](/assets/images/dnschess-sub10C0.png)](/assets/images/dnschess-sub10C0.png){:target="_blank"}

#### sub_1020

One thing to note is that `sub_1020` is a PLT jump function, I had no idea until `@sgtSkid` told me this...  
![plt](/assets/images/dnschess-plt.png)  

When we view where it is referrenced from, it is easy to see that in the `.plt` section it sets the index into the GOT and makes a jump there to referrence whatever is needed.   :1234:  
<br/>

> _Side Note:_ `cs:qword_4010` means a **quad-word** (8-byte) value in a code segment register at that address.   

![plt2](/assets/images/dnschess-plt2.png)

Here we can see that the GOT jumps align nicely with the offsets in the previous screenshot:  
 ![gotplt](/assets/images/dnschess-gotplt.png) 

#### sub_1145
As shown below this function is called by `getNextMove` twice.  
We can see <small style="color:purple;">in the comment</small> that a `"-"` is appended to a `dest` string.   
<small style="color:gray;">(This screenshot will make more sense coming back from reading below...)</small>  
![sub_1145](/assets/images/dnschess-sub1145.png)  

 
 <br/>
   
In `getNextMove` there are two calls to `sub_1145` and also  the resulting string has `.game-of-thrones.flare-on.com` appended at the end.
![sub_1145_used](/assets/images/dnschess-sub1145-used.png)

After we have examined `getNextMove`, we can reliably infer that the `sub_1145` function does the following: (see <small style="color:blue;">blue text</small> in screenshot)
- takes a source string
- appends a dash to it
- appends a alphabet [a-h] to it
- appends a number [1-8] to it  

As the function is called twice, the code block constructs the final string like so:
- **rook**
- rook**-c3**
- rook-c3**-c6**
- rook-c3-c6**.game-of-thrones.flare-on.com**  

This is part of creating a serialised string of the chess piece being placed in a new position, which is made to be comformant to subdomain convention!  

---
 
### getNextMove
What is really interesting is the **`getNextMove`** function, which is essential to the operation of the `ChessUI` program! This function is rather big so I will break it up visually into <u>three sections</u>:  
[![getnextmove](/assets/images/dnschess-getnextmove-big.png)](/assets/images/dnschess-getnextmove-big.png){:target="_blank"}  

#### Section 1

![section1](/assets/images/dnschess-getnextmove-section1.png)
> _Side Note:_ **`fs:28h`** is usually a stack cookie, we can see that at the beginning it is stored in `var_8` and at the end it is xored with itself, this is a _anti stack-smashing_ technique where a mismatch will result in a jmp to the code block that calls `__stack_chk_fail`  :no_entry:  
![cookie](/assets/images/dnschess-stackcookie.png)

The middle bit of this section was previously explored as the string construction code which translates player moves into a hostname format. I won't go into any further detail there :smile:  

<br/>  

What is interesting is at the front of the function right after the epilogue, this section of code is where arguments are passed in: ![fastcall](/assets/images/dnschess-args.png)

From previous posts we might recognise this as standard Linux [fastcall](https://en.wikipedia.org/wiki/X86_calling_conventions#cite_ref-ms_16-1){:target="_blank"} convention, where args are passed in the order of `edi,esi,edx,ecx,r8,r9`

Out of curiousity lets debug the application and figure out what types and what values are passed into the `getNextMove` function...  

![traceargs](/assets/images/dnschess-traceargs.png)

As we can see in the <small style="color:skyblue;">teal section</small>, this code path is the exact same behaviour as seen in the [gif above](./#gif). The <small style="color:red;">red section</small> is the part we are interested in, as these values ultumately end up as args for `getNextMove`...  

<br/>  

We know from `getNextMove` that rsi has to contain a string that correlates to the chessPiece type. However we can see `[rdi+8]`.  
This hints that `rdi` is most likely a pointer to a **<u>struct</u>** of some sort.  

![structing](/assets/images/dnschess-structing.png)

During the debugging process, I moved a chess piece from **D2** to **D3**, then inspected the struct in heap memory:    

![board1](/assets/images/dnschess-board1.png)  

![heapstruct](/assets/images/dnschess-heapstruct.png)

As we can see the index changed from **25** to **26**. After a bit of guesswork, we can now see how the chessboard is laid out in memory. <small style="color:gold">(Yellow = Unknown)</small>  

![board2](/assets/images/dnschess-board2.png)

With this knowledge, we now know how the positions are calculated back in the `sub_1145`/`append_pos` function!  

![mod](/assets/images/dnschess-mod.png)

  
As of now the purpose and identity of **`r8`** and **`edi`** is <small style="color:gold;">unknown</small>...  

<br/>  

In `getNextMove` a global variable `dword_D2120` is written to <small style="color:gold;">`edi`</small>.  
My current guess is that `dword_D120`/`(current_round)` is a round counter.  

- When the game first starts `(startGame)`, the variable is initialised to 0
- After each move is registered this variable is incremented by 1
- This variable is also read and used for some comparison in `getNextMove()`  

![counter](/assets/images/dnschess-counter.png)  

<br/>  

The next mystery is **<small style="color:gold;">`r8`</small>**, it stores `rsp`, which is the stack frame pointer :thinking:  
Tracing through the functions shows that it is passed in to `getNextMove`:  
![r8](/assets/images/dnschess-r8-1.png)

For now we will treat this value as an `array`, and we explore it further in <small style="color:red;">Section 3</small>...  

<br/>  

### [Extra] Discovering more...  

We look at how the `registerMove` function is being called:  
![r8-2](/assets/images/dnschess-r8-2.png)

We see that it is being loaded into `rsi` as an argument for **`_g_timeout_add`**. As `@sgtSkid` points out, this is usually a callback function. Looking at the [documentation](https://developer.gnome.org/glib/stable/glib-The-Main-Event-Loop.html#g-timeout-add){:target="_blank"} shows that it takes three arguments, not only that... this function will call the given function every interval until it returns False.  

Here is a peek at what the code block looks like after I reversed it: :smile:  
![r8-2-reversed](/assets/images/dnschess-r8-2-rev.png)

Through tracing upwards, we can discover that:
- `movePiece` struct is of size **264**
- `[rbp+0]` = `movePiece.old_index`
- `[rbp+4]` = `movePiece.new_index`  
<br/>   
- `dword_D110`/`gameSwitchValue` stores value for a switch table for updating the game's responses  
<br/>   
- `_g_type_cast_ctype_ptr` is actually a saved `gameState` struct with lots of info.   
  - This is evident when the current function is called with two arguments: 
    - The _"fresh"_ gameState returned from a function call
    - A _"stored"_ state  assumed to be from the previous round  
      ![fresh-game](/assets/images/dnschess-fresh-game.png)  

Lets look closer at the functions `strcpy_chessPiece` and `load_chess_images`  

[![chesspiece](/assets/images/dnschess-chesspiece.png)](/assets/images/dnschess-chesspiece.png){:target="_blank"}
When we dynamically reverse the `strcpy_chessPiece` function, we see that the following **`[rdi+92]`** values make no sense when looked at statically.   

We then set a <small style="background-color:red;color:white">breakpoint</small> and look at how the values change across the selection of every single tile:    
![esi-values](/assets/images/dnschess-esi-values.png)

It turns out that this value is a sort of **`chessPieceID`**, that can be used to infer three things:
- Individual pieces of identical types from each other (ie. pawn1 and pawn2)
- Whether the piece is black or white
- What type the piece is

<br/>  
We can prove this by insepcting the not so aptly named `strcpy_chessPiece` and `load_chess_images`  functions ---  
As an example we take the ID of the Rook at H1, it has **uniqueID** `0x17` = **`23`**:  

![chesstype](/assets/images/dnschess-chesstype.png)

<br/>  

#### <small style="color:navy;">Section 2</small>
I got stuck for a fair a bit here but got help from `@sgtSkid` again who pointed out --- What does **`gethostbyname()`** return? 

  
By checking out the [man pages](http://man7.org/linux/man-pages/man3/gethostbyname.3.html#RETURN_VALUE){:target="_blank"}, we see that a [`hostent`](https://docs.microsoft.com/en-us/windows/win32/api/winsock/ns-winsock-hostent){:target="_blank"} is returned,  
or NULL if an error occurs...  

Therefore we know that `var_60` is a pointer to NULL or new **`hostent`** struct:
![b4](/assets/images/dnschess-before-after.png)  
Combining this info with the wireshark analysis earlier, the way we can interact with the program is to have the hostnames resolve to their relevant IP addresses, because the AI will isstantly resign if the resolution fails!  
  
The simple way to do this is by editing the local `/etc/hosts` file like so:
``` 
127.150.96.223        rook-c3-c6.game-of-thrones.flare-on.com
127.252.212.90        knight-g1-f3.game-of-thrones.flare-on.com
127.215.177.38        pawn-c2-c4.game-of-thrones.flare-on.com
127.118.118.207        knight-c7-d5.game-of-thrones.flare-on.com
127.89.38.84        bishop-f1-e2.game-of-thrones.flare-on.com
127.109.155.97        rook-a1-g1.game-of-thrones.flare-on.com
127.217.37.102        bishop-c1-f4.game-of-thrones.flare-on.com
127.49.59.14        bishop-c6-a8.game-of-thrones.flare-on.com
127.182.147.24        pawn-e2-e4.game-of-thrones.flare-on.com
127.0.143.11        king-g1-h1.game-of-thrones.flare-on.com
127.227.42.139        knight-g1-h3.game-of-thrones.flare-on.com
127.101.64.243        king-e5-f5.game-of-thrones.flare-on.com
127.201.85.103        queen-d1-f3.game-of-thrones.flare-on.com
127.200.76.108        pawn-e5-e6.game-of-thrones.flare-on.com
127.50.67.23        king-c4-b3.game-of-thrones.flare-on.com
127.157.96.119        king-c1-b1.game-of-thrones.flare-on.com
127.99.253.122        queen-d1-h5.game-of-thrones.flare-on.com
127.25.74.92        bishop-f3-c6.game-of-thrones.flare-on.com
127.168.171.31        knight-d2-c4.game-of-thrones.flare-on.com
127.148.37.223        pawn-c6-c7.game-of-thrones.flare-on.com
127.108.24.10        bishop-f4-g3.game-of-thrones.flare-on.com
127.37.251.13        rook-d3-e3.game-of-thrones.flare-on.com
127.34.217.88        pawn-e4-e5.game-of-thrones.flare-on.com
127.57.238.51        queen-a8-g2.game-of-thrones.flare-on.com
127.196.103.147        queen-a3-b4.game-of-thrones.flare-on.com
127.141.14.174        queen-h5-f7.game-of-thrones.flare-on.com
127.238.7.163        pawn-h4-h5.game-of-thrones.flare-on.com
127.230.231.104        bishop-e2-f3.game-of-thrones.flare-on.com
127.55.220.79        pawn-g2-g3.game-of-thrones.flare-on.com
127.184.171.45        knight-h8-g6.game-of-thrones.flare-on.com
127.196.146.199        bishop-b3-f7.game-of-thrones.flare-on.com
127.191.78.251        queen-d1-d6.game-of-thrones.flare-on.com
127.159.162.42        knight-b1-c3.game-of-thrones.flare-on.com
127.184.48.79        bishop-f1-d3.game-of-thrones.flare-on.com
127.127.29.123        rook-b4-h4.game-of-thrones.flare-on.com
127.191.34.35        bishop-c1-a3.game-of-thrones.flare-on.com
127.5.22.189        bishop-e8-b5.game-of-thrones.flare-on.com
127.233.141.55        rook-f2-f3.game-of-thrones.flare-on.com
127.55.250.81        pawn-a2-a4.game-of-thrones.flare-on.com
127.53.176.56        pawn-d2-d4.game-of-thrones.flare-on.com  
```
> <u>Side note</u>: The program includes certain optimization features that squishes code together I suspect. IDA makes the wrong assumption about the flow, which threw me off the first couple of times.  
![fakearrow](/assets/images/dnschess-fakearrow.png)


#### Brute force
Before thinking of the clean solution, it is actually possible to filter out all the player moves from the `pcap` file and brute-force the ordering through process of elimination.  This is time consuming but definitely one way to get the flag without having to understand much of the inner workings of the program.  

As an example, I worked my way down the `pcap` file and tried all the moves that seemed to be playable right from the beginning of the match:  
[![wireshark](/assets/images/dnschess-wireshark.png)](/assets/images/dnschess-wireshark.png){:target="_blank"}

Eventually near the very end, I found one move which worked!  
That was **<small style="color:hotpink;">Pawn D2-D4</small>**. With this information, we can actually begin to understand using dynamic analysis what constitutes as a correct and wrong move, since we have a control group for both <u>positive</u> and <u>negative</u> outcome.  

![correct_move](/assets/images/dnschess-correct.gif)

<br/>  

Now we can start to disect the meat of <small style="color:navy;">**Section 2**</small>,  
which is a series of fancy math checks accompanied by early exits: :heavy_plus_sign::heavy_minus_sign::heavy_multiplication_x::heavy_division_sign:  
![check127](/assets/images/dnschess-check127.png)

By setting a breakpoint and using dynamic analysis, we can see what is really being done for the first check:  
![hostent](/assets/images/dnschess-hostent-rax.png)

We inspect the contents at the address `0x5586B2FE36E2` to `0x5586B2FE3710`, we can jump within the Hex view by pressing **"G"** in IDA and passing in the hex address in memory we want:   
![g-for-jump](/assets/images/dnschess-g-for-jump.png)  

After jumping to the address, we see the IP address in string form just above the hostname!  
![real-hostent](/assets/images/dnschess-hostent-revealed.png)  

In the checks, the value used is the IP address in [network byte](https://lwip.fandom.com/wiki/IPv4#Manipulating_addresses){:target="_blank"} order.  

![netbyte](/assets/images/dnschess-netbyte.png)  

The second check tests whether the lowest byte is even or not (using mod 2):
![isodd](/assets/images/dnschess-isodd.png)

The third check is the most fascincating one imo --- It takes the value,  
mods it by 16 and checks if it matches the <u>current round number</u>: 
![mod16](/assets/images/dnschess-mod16.png)  
This is most likely a system to ensure that certain moves are not performed out of order, as this would screw up the flag decryption process  :thinking::game_die:   

<br/>  

With this knowledge, we can analyse all of the IP addresses,  
eliminate non-suitable ones, and order them correctly! 

From the list of mappings above,  
we get a subset by <u>eliminating all that have a lowest byte with odd value<u>:  
```
127.252.212.90        knight-g1-f3.game-of-thrones.flare-on.com
127.215.177.38        pawn-c2-c4.game-of-thrones.flare-on.com
127.89.38.84        bishop-f1-e2.game-of-thrones.flare-on.com
127.217.37.102        bishop-c1-f4.game-of-thrones.flare-on.com
127.49.59.14        bishop-c6-a8.game-of-thrones.flare-on.com
127.182.147.24        pawn-e2-e4.game-of-thrones.flare-on.com
127.200.76.108        pawn-e5-e6.game-of-thrones.flare-on.com
127.99.253.122        queen-d1-h5.game-of-thrones.flare-on.com
127.25.74.92        bishop-f3-c6.game-of-thrones.flare-on.com
127.108.24.10        bishop-f4-g3.game-of-thrones.flare-on.com
127.34.217.88        pawn-e4-e5.game-of-thrones.flare-on.com
127.141.14.174        queen-h5-f7.game-of-thrones.flare-on.com
127.230.231.104        bishop-e2-f3.game-of-thrones.flare-on.com
127.159.162.42        knight-b1-c3.game-of-thrones.flare-on.com
127.53.176.56        pawn-d2-d4.game-of-thrones.flare-on.com
```

After that, we can extract the move order from the third byte:  
```
[4]     127.252.212.90        knight-g1-f3.game-of-thrones.flare-on.com
[1]     127.215.177.38        pawn-c2-c4.game-of-thrones.flare-on.com
[6]     127.89.38.84        bishop-f1-e2.game-of-thrones.flare-on.com
[5]     127.217.37.102        bishop-c1-f4.game-of-thrones.flare-on.com
[11]    127.49.59.14        bishop-c6-a8.game-of-thrones.flare-on.com
[3]     127.182.147.24        pawn-e2-e4.game-of-thrones.flare-on.com
[12]    127.200.76.108        pawn-e5-e6.game-of-thrones.flare-on.com
[13]    127.99.253.122        queen-d1-h5.game-of-thrones.flare-on.com
[10]    127.25.74.92        bishop-f3-c6.game-of-thrones.flare-on.com
[8]     127.108.24.10        bishop-f4-g3.game-of-thrones.flare-on.com
[9]     127.34.217.88        pawn-e4-e5.game-of-thrones.flare-on.com
[14]    127.141.14.174        queen-h5-f7.game-of-thrones.flare-on.com
[7]     127.230.231.104        bishop-e2-f3.game-of-thrones.flare-on.com
[2]     127.159.162.42        knight-b1-c3.game-of-thrones.flare-on.com
[0]     127.53.176.56        pawn-d2-d4.game-of-thrones.flare-on.com
```
After sorting, we then get the move list! 
```
[0]     127.53.176.56        pawn-d2-d4.game-of-thrones.flare-on.com
[1]     127.215.177.38        pawn-c2-c4.game-of-thrones.flare-on.com
[2]     127.159.162.42        knight-b1-c3.game-of-thrones.flare-on.com
[3]     127.182.147.24        pawn-e2-e4.game-of-thrones.flare-on.com
[4]     127.252.212.90        knight-g1-f3.game-of-thrones.flare-on.com
[5]     127.217.37.102        bishop-c1-f4.game-of-thrones.flare-on.com
[6]     127.89.38.84        bishop-f1-e2.game-of-thrones.flare-on.com
[7]     127.230.231.104        bishop-e2-f3.game-of-thrones.flare-on.com
[8]     127.108.24.10        bishop-f4-g3.game-of-thrones.flare-on.com
[9]     127.34.217.88        pawn-e4-e5.game-of-thrones.flare-on.com
[10]    127.25.74.92        bishop-f3-c6.game-of-thrones.flare-on.com
[11]    127.49.59.14        bishop-c6-a8.game-of-thrones.flare-on.com
[12]    127.200.76.108        pawn-e5-e6.game-of-thrones.flare-on.com
[13]    127.99.253.122        queen-d1-h5.game-of-thrones.flare-on.com
[14]    127.141.14.174        queen-h5-f7.game-of-thrones.flare-on.com
```
Follow the move order above, defeat the AI, and get the flag :100::checkered_flag:  
![flag](/assets/images/dnschess-win.gif)

<br/>  

#### <small style="color:red;">Section 3</small>
This section is more of a bonus, just to inspect how the flag decryption process occurs:  
[![decrypt](/assets/images/dnschess-decrypt.png)](/assets/images/dnschess-decrypt.png)
The flag is decrypted <u>two bytes at a time</u>, per round,  
with the 2nd byte of the IP address as the key.  
This key byte is also the only one that isn't subjected to any checks above!  

<br/>  

The `array` variable is passed into the function by referrence, and is populated with values:
- the value of the 3rd byte divided by 16
- the value of the lowest byte in the IP address divided by 2 :man_shrugging:  
- the reply given by the AI  

This is then used in the `make_another_move` code block, which explains the previously unknown stack variables!  
![array_values](/assets/images/dnschess-arrays.png)

<div class="divider"></div>

## What I learned

- Don't overthink the challenge :eyes:  
  - My first instinct was that I needed to beat a computer AI at chess
  - Second guess was somehow I would need to learn all the game mechanics and cheat to win, ie. TP hacks etc
  - In the end its actually super easy. Just replay everythhing in the pcap file to win :cry:  

- `dlsym` means that function importing/exporting is happening:  
   The function calls are then saved to global variables and can be seen referrenced throughout the program  
   ![dlsym](/assets/images/dnschess-dlsym.png)  
   
- Never underestimate dynamic analysis :books:   

- **`G`** for address jumping in IDA

- Network byte ordering from int32

- IDA will actually tell you when **fastcall** is being used, as well as function args:  
  ![fast](/assets/images/dnschess-fastcall.png)

- Completionist approach isn't the best for solving the challenge, but allows you to learn a whole lot!  
