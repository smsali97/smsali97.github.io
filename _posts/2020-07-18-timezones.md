---
layout: post
title: Moment Madness - About time we learnt timezones!
excerpt: "A post where I attempt to discover all things time and why timezones are so mind-boggling and how can they be handled"
modified: 2013-07-19
tags: [programming, timezones, date, opinion]
comments: true
pinned: true
image:
  feature: timezones.png
  credit: Eric Muller
  creditlink: http://efele.net/maps/tz/world/)

share: true
---
 

If you ever had to deal with anything remotely related to time in programming especially if it involves timezones, you know what an eyesore it is.

![](frustrated.gif)

The New York Times wrote an excellent [pitch](https://www.nytimes.com/2016/11/06/opinion/sunday/time-to-dump-time-zones.html?_r=0) to ditch timezones altogether and adopt UTC instead. If we were to go ahead, it will be a fundamental psychological shift in which our biological clock will learn to accept that 12 pm doesn't mean morning but rather learn to adapt to with the sun as we were used to doing so centuries ago. 

But until that doesn't happen,  we will be stuck with time zones for now.

# So What even are Timezones?

First of all, lets clear the confusion out, are timezones offsets? Well, yes. But *no*. Let me explain. Timezones are regions of land that correspond to places in the world in which the local time is the same across the region, these zones have been laid out in special maps. Like the ones on the image.

![timezones.png](timezones.png)
{: .pull-right}

Why have we done this? This is because we wanted to have a semantic meaning attached to the times we see in our lives. 12 o'clock means noon, 4 o'clock means afternoon and so on. All of these periods have a particular meaning associated with sunlight.  But in order to achieve this, and knowing the shape of the earth allows only some parts to face the sun, we will need to piece the earth up into portions, and hence timezones are born.

To represent timezones we use the timezone name with the corresponding offset which has the number of hours the zone is ahead or behind with. With the reference point being UTC or Coordinated Universal Time. For example *'Asia/Singapore'*  has an offset for UTC+8 currently. So a 12 pm in UTC time corresponds to 8 pm in local time in Singapore. 

But what about exceptions like Daylight Savings Time and its related counterparts? Just to clear out the confusion, EDT and DST arent timezones per se. We use those in the same contexts interchangeably as if they belong to the same category but they are not the same. They are simply some *rules* we have set up that apply to some timezones and as a result change the time as well. The bottom line is that the offset changes, but the underlying timezone will stay the same. But once the rules' time elapses we reset our clocks for those timezones only and the offset comes back to the original value.

One last bit to add more confusion into the mix, time-zones, and offsets are a **many-to-many** relationship! One timezone can have many offsets (depending on whether there is Daylight Savings, etc) but similarly, one offset can also belong to many timezones as well! For example, the UTC+02:00 has a lot of countries that fall in it like Bulgaria, Russia Egypt, and the UK! So it's not particularly uncommon for countries to share timezone offsets.  

For a full detailed read on this, please check out the article [here](https://spin.atomicobject.com/2016/07/06/time-zones-offsets/)!

Here are some general recommendations and tips I have that you should consider when working with timezones:

> Also for the code snippets below, I will be using Moment.js for demonstration purposes but you can easily find the appropriate date-time library for your favorite language. Just to be sure to use any popular one, and not reinvent the date-time wheel, there is a reason all of these calculations are so complex.
`const moment = require('moment-timezone'); // import moment`

## Tip #1: Work with UTC internally.

If it hasn't been obvious yet, I am a strong proponent of having a standardized format throughout. It will make your life easier and give you plenty of sleep at night. Especially if the business is spread across different countries or intends to expand. No more pesky time conversions and inter-time zone formatting, one format plain and simple. 

One nice rule of thumb you can have is that you should do everything in UTC, and at the last second convert it to to the local timezone for display purposes only

{% highlight javascript %}
moment.tz.setDefault("UTC") // ideal first step, set timezone to utc or configure it manually
const currentTime = moment() // get current time in local time (Currently in UTC or GMT+0) 
const timeInSingapore = currentTime.tz("Asia/Singapore") // convert time to Singapore
const timeToStoreInDB = timeInSingapore.utc() // store time in utc
{% endhighlight %}

## Tip #2: However, If you want to work with local times, store the offset, *not* the timezone.

 Another alternative you can opt for is store precisely what the local time was at that specific moment, with a time offset to UTC. This allows both the date and time to be identified at that instantaneous moment and can be verified later.  But one thing I'd recommend is never storing the timezone string literally. Because remember timezones aren't reliable. Their offsets change, if you try to dig up an old local time value and use the timezone to get the appropriate time, you will face the following issues:

- You will need to do some additional bookkeeping to store the timezone internally (which is going to result in duplication and never a good practice)
- Need to make sure that whether daylight time savings was applicable and have separate flags for that as well.
- Business logic that handles this must be robust enough to consider all these cases

The bottom line,  almost all databases have robust support for date-time formats, use that instead, rather than opting for meta-attributes and making your logic unnecessarily complex. 

## Tip #3: For standardized formatting stick to the ISO 8601.

They are around hundreds of formats listed on Wikipedia. But by far the most popular is the ISO 8601. That's it. For its merits, it follows a logical progression from higher granularity to lower *(Year → Month → Day → Hour → Minute → Second → Zone)* and it makes sorting much easier. 

{% highlight javascript %}
const completeTimeStamp = currentTime.toISOString() 
// Should Output '2020-07-12T09:04:16.217Z'
{% endhighlight %}

Also, a PSA by XKCD advocates it too:

![xkcd.png](xkcd.png)

##### These are my opinions, and you can differ and should! Everyone thinks about time differently. After all, if you need any proof that the entire world can never come into a universal agreement, just look at the time.  

### One small caveat:

I have been advocating UTC as the panacea for all your timezone troubles but the truth is the earth rotation's speed time and UTC can sometimes drift apart, hence a leap second needs to be added or removed to UTC to bring them back together. Read this article on the [American Scientist](https://www.americanscientist.org/article/the-future-of-time-utc-and-the-leap-second) if you want to know why this happens. So for 99% of use-cases, UTC will work just fine but it is not the most reliable source of truth down to the very millisecond.
Also as a helpful glossary for all things time here is the [one](https://unix4lyfe.org/time/) posted by the unix4lyfe blog
{: .notice} 