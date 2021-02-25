---
layout: post
title:  "[LINE@]Auto Registration"
date:   2018-08-16 23:15:20 +0900
categories: LineAt
---
Hello.  
I found something interesting on LINE@.  
LINE gives us some regulations in normal LINE accounts to prevent abuse.  
But LINE@ has few regulations.

## What is LINE@
LINE@ is a LINE account for businesses.  
__LINE@ has been changed to "LINE Official Account" in April 2019.__  

## Motivation
[add this section on February 25th, 2021.]  
On August 4, 2018, a bot caused a stir in the world.  
The name of the bot is "オバマbot"(Obama bot).  
Obama means the 44th President of the United States.  
The "Obama bot" started working around August 4th, the day of his birthday.  
The Obama bot sent the message   
__"Happy Birthday to Me (cake emoji)"__   
to an unspecified number of users via LINE, and added itself to their friend list without permission.  
And it's not just one.   
Some people found themselves with 3, 4, or even more than 10 "Obama bots" lined up in their friends list.
<br>
<br>
I was intrigued by this incident and began to investigate the LINE@ system that the Obama bot was using.  
In the process, I found a way to create a large number of accounts automatically.

## Auto Registration
Today, I tried to register LINE@ accounts automatically and it worked.    
I published [tutorial](https://github.com/k0tayan/LINEAtAuto).  
~~Make your LINE@ great.~~  
LINE@ is already closed.

<br>
<img src="https://user-images.githubusercontent.com/16555696/44211704-94535d80-a1a4-11e8-9225-65285cabe045.PNG" alt="Project Sekai Custom Beatmap Loader 2" width="400"/>
<br>

## Why is this possible ?
Normally, only 10 LINE@ accounts can be created for a single LINE account.  
However, the account deletion did not work properly, and the deleted account continued to remain.  
Therefore, I was able to create a large number of accounts by these steps.  
1. creating an account
2. doing some arbitrary processing
3. and then deleting the account.