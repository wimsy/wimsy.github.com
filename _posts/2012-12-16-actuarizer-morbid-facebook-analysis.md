---
layout: post
title: Actuarize to Creep Out Your Friends with Actuarial Tables
published: false
tags: [fql, facebook, xkcd, python, api, actuary, flask]
comments: true
---

# Actuarize to Creep Out Your Friends with Actuarial Tables

I don't know. I guess I was feeling a bit morose this week. Nah, actually I've wanted to do this ever since I came across [xkcd](http://www.xkcd.com)'s [clever use of actuarial tables](http://blog.xkcd.com/2012/07/12/a-morbid-python-script/). Also, I wanted an excuse to try out the [Facebook Open Graph API](https://developers.facebook.com/docs/reference/apis/). 

So, I'm going to use cold, hard statistics to tell my friends when they can expect to die. Not any particular friend, mind you, but some statistical sampling of them. I even worked with my friend [Scott](https://github.com/scottlnorvell) to whip up a little web app that anyone can do to get the same basic stats that xkcd pulled together.

## What I learned
- I had my first ugly experience with [Facebook's OAuth process](https://developers.facebook.com/docs/howtos/login/login-for-desktop/). I'm not going to say that I exhausted it (or even learned what I was doing wrong) but it was painful nonetheless. I think if I ever get serious about using Facebook data, I'll need to revisit this at length.
- After my failed attempt at Facebook's OAuth, I punted and chose to use [Temboo](http://www.temboo.com) to accomplish the login task. They provide the necessary Callback URL that I found so challenging the first time around (the idea for a web app came later).
- I used the standard [Facebook Python SDK](https://github.com/pythonforfacebook/facebook-sdk/) with [FQL](https://developers.facebook.com/docs/reference/fql/) for the data requests themselves.
- [Scott](https://github.com/scottlnorvell) adapted xkcd's [actuary.py script](http://blog.xkcd.com/2012/07/12/a-morbid-python-script/) to receive input from downloaded Facebook data and return the standard response.
- I took my first run at building something on the [Flask](http://flask.pocoo.org/) web framework, along with [Flask Bootstrap](https://github.com/mbr/flask-bootstrap) for formatting.

It's really amazing how quickly you can explore new tools and techniques when you're exploring.  Most of the tools listed above were not on the agenda, but they seemed to solve the problems I had or enable us to take the project a step further. Also, I learned that [Jekyll](http://jekyllrb.com/) doesn't handle GitHub-flavored markdown out of the box.

The basic code for downloading data and processing it is [here](https://github.com/wimsy/actuary), lightly documented (at best) as usual.

## The results
When I finally downloaded the data and calculated ages, my 491 or so friends dwindled to 144 in the analysis population. I don't know if anything about a person who would suppress a birth year (age) that would necessarily skew the data, but that is certainly a source of potential error. 

However, out of that population I was able to derive the following statistical predictions:

<pre>
There is a 5%  chance of someone dying within 0.12 years (by 2013).
There is a 50% chance of someone dying within 1.66 years (by 2014).
There is a 95% chance of someone dying within 5.94 years (by 2018).

There is a 5%  chance of everyone dying within 67.17 years (by 2080).
There is a 50% chance of everyone dying within 72.06 years (by 2085).
There is a 95% chance of everyone dying within 78.28 years (by 2091).

Probability of all dying in 1.0 year: &lt;0.001%
Probability of a death within 1.0 year: 33.16%
</pre>

It's kind of scary to think someone I care about has a 50% chance of bing gone in 20 short months. It's also hard no to ascribe that to a specific person in my mind, although that's not strictly valid, given that these numbers reflect the aggregate risk of accidents, premature illness and a whole host of age-related death-dealing ailments.