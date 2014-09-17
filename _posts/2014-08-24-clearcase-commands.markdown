---
layout: post
title: Clearcase common used commands
category: idev
tags: clearcase idev cmd
year: 2014
month: 08
day: 24
published: false
customid: 20140824_CLEARCASE_CMD
summary: some common used clearcase commandlines got from my daily work.
image: idev/ubuntu-l2tp-ipsec.png
---
[TOC]
#List All Checked files#
`ct lspr -co`

#Add an file to vobs#
```bash
ct co -nc .                         # checkout current dir
ct mkelem -nc filename              # Add element
```
then create an branch and edit the file

#find version#
`ct find ../../ -version "brtype(HS60407_2Q)" -print`

#change file permission#
`ct protect -chmod +x file`