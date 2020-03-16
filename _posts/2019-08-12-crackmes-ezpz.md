---
layout: post
title: Crackmes - ezpz
categories: ['writeup']
date: 2019-08-12
---


> I was helped on this one because I'm a <del>fucking retard</del> pleb  
I didn't `copy+paste` the password and repeatedly typed it in wrong :man_facepalming:

## Resources/Tips
- [Link](https://www.crackmes.one/crackme/5d295dde33c5d410dc4d0d05){:target="_blank"} to challenge
- Disassemble on Windows 64-bit using IDA :pray:
- [Reversing for beginners](https://beginners.re/RE4B-EN.pdf){:target="_blank"} _(Coutersy of `@SeargantSkid`)_

<div class="divider"></div>

## Main flow of the program

![prompt.gif](/assets/images/ezpz-start.gif)

- print(prompt)
- get(username)
- compare(username)
   - <small style="color:green">Continue</small> if comparison succeeds
   - Else <small style="color:red">fail and die</small>
- print(prompt)
- get(password)
- compare(password)
   - <small style="color:green">Continue</small> if comparison succeeds
   - Else <small style="color:red">fail and die</small>
- print(:checkered_flag:)


<div class="divider"></div>

## tl;dr
- Disassemble the program
- The username and password in plaintext is laid right infront of you
- Figure out the control flow to find out why the program early exits...  

![up](/assets/images/ezpz-up.png)

<div class="divider"></div>

## New concepts
It was easy to get thrown off by:
- 
The source file being **`C++`**, which probably looked like this:
  
    ```c++
    #include<iostream>   

    using namespace std;   

    int main() { 
        cout<<"Hello World"; 
        return 0; 
    } 
    ```
    instead of this:
    ```c
    #include <stdio.h>  

    int main() {    
        printf("Hello World");    
        return 0;   
    } 
    ```
         
<br/>  
  
- the different calling [conventions](https://en.wikipedia.org/wiki/X86_calling_conventions#cite_ref-ms_16-1){:target="_blank"}, 64-bit PE uses `rcx,rdx,r8,r9` during fastcall which differs from the x64 convention on Linux which uses `rdi,rsi,rdx,rcx,r8,r9`...  
  
- Also the function name _**mangling**_ --- like who the fuck calls `scanf` as **`_ZStrsIcSt11char_traitsIcESaIcEERSt13basic_istreamIT_T0_ES7_RSbIS4_S5_T1_E`**  
 
- C++ uses _"vtables"_ for classes, which becomes very complex to understand in the disassembly <small style="font-size:12px">(especially when first starting out like me!)</small>

<div class="divider"></div>

## Learning new things 

Some of this knowledge I got from `@SeargantSkid`  :sunglasses:   

<br/>  

- `cin` and `istream` can be thought of as `stdin`
- `cout` and `ostream` can be thought of as `stdout`
- Pay attention to the **operator** symbol in the function comments

![operators](/assets/images/ezpz-op.png)

The first red block can be identified as a comparison function (`==`) even though the function name is mangled, also notice how all the **ostream** functions have the arrows in the left direction?  

  
When using `stdin`/`stdout`:  
- You first have to initialize a stream
- Then whenever you need to use `cout`, you allocate the stream to `cout`
- From there you pass your data and the stream together to a function that will print to stdout

<div class="divider"></div>

## Google is your best friend _(sometimes)_ :mag:  

Based on the application logic itself I knew it was some form of comparison:
![compare](/assets/images/ezpz-comp.png)

But by throwing the function name in Google, I found several sites that do some code coverage analysis or something... but when I looked up the actual function the [_results_](http://sbt.science.uva.nl/bterwijn/coverageReport/usr/include/c++/4.8/bits/basic_string.h.gcov.html#2491){:target="_blank"} were amazing!  

![code_coverage](/assets/images/ezpz-lcov.png)


The code seems to be in `/usr/include/c++/4.8/bits/basic_string.h`:    

![sauce](/assets/images/ezpz-sauce.png)

Granted this does not work with every mangled function name, but its always worth a shot!  
