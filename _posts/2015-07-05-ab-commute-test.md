---
title: "Commute AB Test"
author: "Michael Wimsatt"
date: "July 5, 2015"
output: html_document
layout: post
published: true
tags: [R, experiment, commute, map, leaflet]
comments: yes
---

# Is the light worth the time?

My evening commute is stressful enough (around an hour, with a fair share of traffic). But the most stressful part comes in the first five minutes - where I need to turn left into a heavily used two-way street. If you're a driver you probably know what i mean. Never fun. Every day? A pain.

Of course, there's a solution to this dilemma: a traffic light. Only one block away is a convenient traffic light where I can leisurely wait my turn and enter traffic without peeling out or raising my heart rate. The problem is that I have to take a little roundabout route to get there. And I just don't know if it's worth the extra time and effort.

![Map of two routes](/images/2015-07-05-ab-commute-test/route_map.png)

<sup><a href="https://thenounproject.com/term/stop-sign/51713/">Stop sign</a> and <a href="https://thenounproject.com/term/traffic-light/53685/">traffic light</a> icons by Jonathan Li from <a href="http://thenounproject.com">the Noun Project</a>.</sup>

### Hypothesis

My hypothesis is that the longer (distance) trip is longer (time) on average, but more predictable, and so maybe not even all that much longer. Hence, taking the low-stress route is not as costly as I might imagine.

### The experiment

So, taking this problem and having recently learned that [real scientists make their own data](http://seanjtaylor.com/post/41463778912/real-scientists-make-their-own-data), I decided to conduct some randomized trials on my daily commute. I randomly selected a route each time I left the office and recorded the time it took to complete that route.



Over the course of five months I recorded 53 trials. Here's a summary of the results from those trials:


{% highlight text %}
## Source: local data frame [2 x 4]
## 
##   route  n median.time   st.dev
## 1  long 31   104.40968 19.14317
## 2 short 22    70.78513 38.14085
{% endhighlight %}

![plot of chunk unnamed-chunk-3](/images/2015-07-05-ab-commute-test/unnamed-chunk-3-1.png) 

### Takeaways

- The median time difference is 34 seconds. 
- There is, indeed, greater volatility in times for te short route.
- Even with volatility half the time the short route is faster than even the fastest trip on the long route.
- However, by far the worst time was associated with the long route.

For me, it's a pretty simple tradeoff. Avoiding the angst of that intimidating turn is worth 34 seconds per trip. I think that's what I'll start doing.

I realize that the amount of time I spent building the data recorder, conducting analysis and writing this post blows away many times the time differences we're talking about here, but this is about trying new things. Hopefully someone out there can do something truly meaningful with tools like these.

### How I recorded the data

Using [Pythonista](http://omz-software.com/pythonista/) and [Workflow](https://workflow.is/), the app randomly selects a route each day and presents a stopwatch which I start at the fork and stop at the finish. This is automatically appended to an [Evernote](http://www.evernote.com/) note. Whatever I could document about the code is available [on GitHub](https://gist.github.com/wimsy/bfaf960e35de9eacb721). The analysis can be found in a [large repository](https://github.com/wimsy/mileage_analysis) I used for other driving-related analysis and posts.
