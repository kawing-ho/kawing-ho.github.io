---
layout: post
title: 0x00sec Discord Re
categories: ['life']
date: 2020-02-26
---


> Here's some things I learned from watching the [**`0x00sec`**](https://0x00sec.org/){:target="_blank"} livestream.  
(The challenge being reversed was [`applestore`](https://pwnable.tw/challenge/#7){:target="_blank"}) :apple:  

<div class="divider"></div>

#### Global Variables
They can be seen in IDA because they appear with something like:  

~~~ sql
mov eax, offset myCart
~~~

<br/>
  
### STRUCTS


#### IDA Structs Tab
Can be used to actually define a struct  in IDA which will help it <small style="font-size:9px">(and you)</small> recognize field referrences in the remaining disassembly! 

In this case it was used to create an <small style="color:brown">ITEM</small> struct:
```c
typedef struct _item ITEM {
   char * name;
   int price;
   ITEM *next;
   ITEM *prev;
}
```

The way to call this is:
- Change Fieldtype
- Offset 
- Struct
- ITEM*  

#### Assembly dealing with structs

Something to note is that when referrencing structs:
```c
# Assuming all fields have size of 4 bytes here  

# eax is the address of an ITEM struct  

mov edx, [eax] -> field1    <---- this is not the whole struct!  
mov edx, [eax+4] -> field2
mov edx, [eax+8] -> field3
```

<br/>

### Linked lists

You can tell when the struct has 4/8 bytes and is referrenced in a way which doesnt quite fit other data types ie. `str`, `int`. This can be identified by `offset + 8`

```
_mycart = _mycart[8]
```

<br/>  

### lea is fancy mov

In cases where optimization is turned on for the compiler. A statement such as `var++` will be done with a `lea` instruction compared to the usual `mov`, which saves one or two lines of instructions...  :thinking:  
