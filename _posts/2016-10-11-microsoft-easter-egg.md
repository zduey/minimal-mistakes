---
title: Microsoft Easter Egg
header:
  - image: easter-egg.png
categories:
  - Gotcha
tags:
  - python
  - excel
  - dates
---


**tl;dr 1900 was not a leap year and you know it Microsoft** [Gist](https://gist.github.com/zduey/528e78430b6ae8107ddd05f5752dff77) 

Ever watched a Pixar movie, or more likely, read a review after the fact outlining all of the references to other Pixar movies? Well, this is not exactly like that, but you can think of this as a self-referential Easter egg between past and present Microsoft Office products. It more typical parlance: a festering bug.

A while back, I was writing a script to do some pretty simple data processing. As I was building the script, I noticed a very odd problem: I had data for weekends when there should not have been. Fast forward a frustrating hour to the point where I discovered that what I thought was my coding error or some quirk of the data was the result of one easy to fix problem, one very random trivia fact, and one very old bug in the Microsoft ecosystem.

The problem started with my need to convert between an Excel serial date and a datetime format understood by Python. As a first pass, I googled around and found some starter code, which seemed like it would do the trick:

```python
import datetime

def xlserialdate_to_datetime(xlserialdate):
    excel_anchor = datetime.datetime(1900, 1, 1)
    delta_in_days = datetime.timedelta(days=xlserialdate)
    converted_date = excel_anchor + delta_in_days

    return converted_date
```

Unfortunately, running a quick test on the serial date for January 1, 2016 showed this was not  doing what is was supposed to. Instead of the January 1, 2016 I was expecting, I got back January 3, 2016. My first guess was that it (at least partially) had something to do with indexing. Python is 0-based while some languages start at 1. Easy problem solved. Another google searched confirmed my guess and also revealed that serial dates are pegged to January 1, 1900. If you are new to the computing world, this may sound as good a place as any to start. However, the vast majority of other date systems use a totally different base unit: January 1, 1970 also known as the [unix epoch](https://en.wikipedia.org/wiki/Unix_time) 


Still needing to find out why I was off by a day, I decided to use my nascent understanding of search algorithms starting with the fact that the error had to occur somewhere between January 1, 1900 and January 1, 2016 (way to go… really narrowed it down there). Using a manual binary search algorithm, I found that the difference began on February 29, 1900. For Python, this day does not exist, so the equivalent serial date was actually March 1, 1900.

In my mind, 1900 % 4 = 0, so this was clearly a Python datetime module problem. I was already planning the github issue I would submit and the follow up PR, but decided to do another quick google search just to be sure. A second surprise followed: 1900 is in fact not a leap year! This was definitely not what I learned in school (or if I did, I promptly forgot it). Not only must a year by divisible by 4, but it must also be divisible by 400. There is your [winning trivia fact for next Tuesday evening](http://science.howstuffworks.com/science-vs-myth/everyday-myths/question50.htm)

Finally having narrowed the problem down to a specific day and location, I was curious how this quirk in Excel was something I had not heard about before. So, I went back to the google machine and began looking for some mention of this issue to see when it would be fixed. What I found was the third surprise of the afternoon and also the most frustrating. The Microsoft support response to someone who brought up this issue was essentially: [eh… we know about the problem, but it affects too many things for us to actually fix it]( https://support.microsoft.com/en-us/kb/214326)

Maybe its just me, but if I used this excuse with my manager or some user of the code I write, I would be immediately left with a scornful look and sufficient reproach that I would go back and devote however time was necessary to fix the underlying problem.

Getting back to the code, I ended up with the following code snippet for converting between serial dates and datetime objects:


<script src="https://gist.github.com/zduey/528e78430b6ae8107ddd05f5752dff77.js"></script>