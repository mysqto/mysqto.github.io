---
layout: post
title: kepbod color scheme for mintty
category: idev
tags: mintty color scheme xshell
year: 2014
month: 08
day: 17
published: true
customid: 20140817_kepbod_mintty
summary: A clone of kepbod color scheme for mintty.
---

I'm using xshell under windows for linux remote access and cygwin and msys for some linux/unix features as well. As xshell, [kepbod color scheme](https://github.com/kepbod/colour_kepbod) is the best choice. So I port it for mintty use.
```bash
ForegroundColour    =   197,200,198
BackgroundColour    =   39,40,34
CursorColour        =   253,157,79
Blue                =   102,217,239
BoldYellow          =   244,191,117
BoldWhite           =   249,248,245
BoldCyan            =   18,207,192
Green               =   166,226,46
Magenta             =   174,129,255
BoldMagenta         =   225,163,238
BoldBlue            =   111,194,239
Red                 =   251,159,177
BoldBlack           =   56,56,48
Black               =   39,40,34
BoldGreen           =   172,194,103
BoldRed             =   222,175,143
Yellow              =   253,151,31
White               =   245,244,241
Cyan                =   161,239,228
```
Copy the code and append it to the .minttyrc(creat one if no this file) in your cygwin or msys home directory. Enjoy your hacking.
![mintty kepbod](/img/20140817/kepbod_mintty.png "kepbod for mintty")
