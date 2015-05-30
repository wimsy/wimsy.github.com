---
title: "What's My Rush Hour?"
author: "Michael Wimsatt"
date: "May 30, 2015"
output: html_document
layout: post
published: true
tags: [R, mileage, automatic, Python]
comments: true
---

# What's My Rush Hour?

This is my second post about the data I'm getting from the 
[Automatic](https://www.automatic.com/) device and app I use daily with my car. 
In the 
[last post](http://www.quantary.net/2015/02/21/validating-automatic-mileage-data.html) 
I tried to assess the accuracy of the mileage and fuel usage data Automatic was 
reporting. It turns out it's pretty accurate. Any discrepancies appear to come 
when the app is unable to connect to the device for some reason. That was common 
early on, but less so now. 

In this post, I'm trying to answer one very specific question: What are my rush 
hours? How early do I need to leave to "beat" rush 
hour? Within rush hour, are some times better than others?

## About my commute

My 43-mile commute is sort of a "reverse" commute. I live in Queens, NY, and 
work in Suffolk County, Long Island. Since I'm commuting out of the city, I face
less traffic than people going in the opposite direction, but I still deal with 
significant traffic at certain spots. I tend to hit more traffic coming home, 
but it drops off significantly later in the evening so - on average - my 
morning and evening commutes are both around one hour.

On a typical day I arrive around 10AM and leave at 6:30 or 7:00.

## Commute times and rush hour



The first thing I wanted to see was whether my impression that evening commutes 
are generally longer and less predictable is correct. This distribution of 
commute durations by morning and evening confirms that. As you can see, the 
evening commmute has a much higher density above 60 minutes, extending frequently
above 80. 

![plot of chunk unnamed-chunk-2](/images/2015-05-30-whats-my-commute/unnamed-chunk-2-1.png) 

So, what is my rush hour? Well, first I'll define rush hour for me. I'll call it 
the **period of time during which the expected time is greater than 60 
minutes**. If you look at the plot of durations below against departure time for
morning and evening commutes, you can see a regression curve with confidence 
regions. 

![plot of chunk unnamed-chunk-3](/images/2015-05-30-whats-my-commute/unnamed-chunk-3-1.png) ![plot of chunk unnamed-chunk-3](/images/2015-05-30-whats-my-commute/unnamed-chunk-3-2.png) 

From this, my conclusion is that morning rush hour starts for me somewhere 
between (extrapolating) 6:30 AM and 7:30 AM and ends between 8:30 and a little 
after 9:00. Similarly my evening rush hour begins between 2:30 (!!!) and 3:30 
and lasts until somwehere between 6:30 and 7:30. It turns out that the common 
wisdom around my office that you need to leave before 2 if you're going to New 
Jersey is (roughly) true!

## A little data cleaning

In an effort to tighten up my estimates a bit, I'd like to focus on "normal" 
commutes. For each trip, I also have path information, and my commutes aren't 
always direct. For example, sometimes I drop my partner off at work or pick her 
up on the way home. 

I decided to limit the analysis to commute paths that followed one of the top 
three routes for each direction. The top 3 routes represent 83% and 64% of trips 
for the morning and evening commutes, respectively. 



Here are the morning and evening commute profiles for just the top three routes 
in each direction.

![plot of chunk unnamed-chunk-5](/images/2015-05-30-whats-my-commute/unnamed-chunk-5-1.png) ![plot of chunk unnamed-chunk-5](/images/2015-05-30-whats-my-commute/unnamed-chunk-5-2.png) 

With this cleaner data set, you can see that the rush hour windows are 
reduced. Summarized...

| Commute | Start          | End            |
| ------- | -------------- | -------------- |
| morning | 6:40 - 7:20 AM | 8:40 - 9:00 AM |
| evening | 2:50 - 3:30 PM | 6:10 - 6:40 PM |

### A note about routes

I filtered my analysis to four routes that comprise the top three routes in each
direction. 


{% highlight text %}
##                          
##                           AM PM
##   CIP-LIE                 77 41
##   All LIE                 20 21
##   GCP-NSP and LIE to Sag  11  0
##   NSP to 37A then LIE-CIP  0 20
{% endhighlight %}



**CIP-LIE - 43.0 miles**

![CIP-LIE map](/images/2015-05-30-whats-my-commute/map0002.png)

**All LIE - 41.8 miles**

![All LIE map](/images/2015-05-30-whats-my-commute/map0005.png)

**GCP-NSP and LIE to Sag - 44.8 miles**

![GCP-NSP and LIE to Sag map](/images/2015-05-30-whats-my-commute/map0025.png)

**NSP to 37A then LIE-CIP - 44.8 miles**

![NSP to 37A then LIE-CIP map](/images/2015-05-30-whats-my-commute/map0010.png)

I use the navigation app [Waze](https://www.waze.com/) on all my trips, and 
usually follow its route recommendations based on traffic and other incidents. 
So, my route is theoretically "optimized" for the shortest duration. As a 
result, a pattern emerges:

![plot of chunk unnamed-chunk-8](/images/2015-05-30-whats-my-commute/unnamed-chunk-8-1.png) ![plot of chunk unnamed-chunk-8](/images/2015-05-30-whats-my-commute/unnamed-chunk-8-2.png) 

Here you can see that my primary route included the Long Island Expressway (LIE)
and the Cross Island Parkway (CIP) in both directions. Second to that, came the
most direct route (all on the LIE) - typically outside of rush hour. 

Finally 
there were the tertiary routes, which both involved the Northern State Parkway 
(NSP; Queens in the morning and Suffolk in the 
evening). In the morning commute, this route seemed to come into play 
at the end of rush hour. In the evening, it looks like more of a toss up
with the primary route. I can confirm that when I would look at route options in 
Waze the times were often within a couple of minutes of each other.

## Conclusion

My commute is pretty predictable in the morning and the rush hours make sense. 
In the evening, rush hour starts earlier than I would have guessed - 
and trying to get out a little before 5:00 to "beat traffic" is the worst thing 
I can do. Rush hour is longer in the evening (no surprise), but ends a bit 
earlier than I would have guessed.

Also, the times involved here are interesting. While it *feels* like I regularly 
sit on the highway for close to two hours in my evening commute, the difference 
between peak rush hour and cruising is about 20 minutes, on average. Of course
the distribution is wide, and there are definitely some painful, painful 
exceptions. 

Commuting sucks. Thanks for nothin' Robert Moses. 

### Data and tools

The standard data dump from the Automatic dashboard is a CSV file with one row 
per trip. Information about each trip includes beginning and ending times and 
locations, the route traveled coded as a 
[polyline](https://developers.google.com/maps/documentation/utilities/polylinealgorithm), 
and some data related to gas mileage. For this, I focused on times, locations 
and routes. 

Statistical analysis was conducted in RStudio with lubridate, plyr, dplyr, 
ggplot2, fields and qcc libraries. I did the path analysis in an iPython 
notebook using a Python script from 
[signed0](https://gist.github.com/signed0/2031157). 
The code and related files can be found 
[on GitHub](https://github.com/wimsy/mileage_analysis). 

