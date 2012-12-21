---
layout: post
title: Actuarize to Creep Out Your Friends with Actuarial Tables
published: true
tags: [fql, facebook, xkcd, python, api, actuary, flask]
comments: true
---

# Actuarize to Creep Out Your Friends with Actuarial Tables

So, now that we've all survived the [end of the world](http://en.wikipedia.org/wiki/2012_phenomenon), it feels like we could use a statistical discussion of our collective mortality. Right? No? Actually I've wanted to do this ever since I came across [xkcd](http://www.xkcd.com)'s [clever use of actuarial tables](http://blog.xkcd.com/2012/07/12/a-morbid-python-script/). Also, I wanted an excuse to try out the [Facebook Open Graph API](https://developers.facebook.com/docs/reference/apis/). 

So, I'm going to use cold, hard statistics to tell my friends when they can expect to die. Not any particular friend, mind you, but some statistical sampling of them. I even worked with my friend [Scott](https://github.com/scottlnorvell) to whip up a little web app that anyone can do to get the same basic stats that xkcd pulled together.

## What I learned
- I had my first ugly experience with [Facebook's OAuth process](https://developers.facebook.com/docs/howtos/login/login-for-desktop/). I'm not going to say that I exhausted it (or even learned what I was doing wrong) but it was painful nonetheless. I think if I ever get serious about using Facebook data, I'll need to revisit this at length.
- After my failed attempt at Facebook's OAuth, I punted and chose to use [Temboo](http://www.temboo.com) to accomplish the login task. They provide the necessary callback URL that I found so challenging the first time around (the idea for a web app came later, so setting up a webpage seemed like overkill for some basic number crunching).
- I used the standard [Facebook Python SDK](https://github.com/pythonforfacebook/facebook-sdk/) with [FQL](https://developers.facebook.com/docs/reference/fql/) for the data requests themselves.
- [Scott](https://github.com/scottlnorvell) adapted xkcd's [actuary.py script](http://blog.xkcd.com/2012/07/12/a-morbid-python-script/) to receive input from downloaded Facebook data and return the standard response.
- I took my first run at building something on the [Flask](http://flask.pocoo.org/) web framework, along with [Flask Bootstrap](https://github.com/mbr/flask-bootstrap) for formatting.

It's really amazing how quickly you can discover new tools and techniques when you're exploring.  Most of the tools listed above were not on the agenda, but they seemed to solve the problems I had or enable us to take the project a step further. 

The basic code for downloading data and processing it is [here](https://github.com/wimsy/actuary), lightly documented (at best) as usual.

## The results
When I finally downloaded the data and calculated ages, my 491 or so friends dwindled to 146 in the analysis population. I don't know if anything about a person who would suppress a birth year (age) that would necessarily skew the data, but that is certainly a source of potential error. 

However, out of that population I was able to derive the following statistical predictions:

<pre>
There is a 5%  chance of someone dying within 0.12 years (by 2013).
There is a 50% chance of someone dying within 1.64 years (by 2014).
There is a 95% chance of someone dying within 5.88 years (by 2018).

There is a 5%  chance of everyone dying within 67.26 years (by 2080).
There is a 50% chance of everyone dying within 72.11 years (by 2085).
There is a 95% chance of everyone dying within 78.3 years (by 2091).

Probability of all dying in 1.0 year: &lt;0.001%
Probability of a death within 1.0 year: 33.50%
</pre>

It's kind of scary to think someone I care about has a 50% chance of being gone in 20 short months. It's also hard no to ascribe that to a specific person in my mind, although that's not strictly valid, given that these numbers reflect the aggregate risk of accidents, premature illness and a whole host of age-related death-dealing ailments.

## Lifetime footprints
I also wanted to plot the lifetime footprint of my current Facebook friends a la [xkcd's astronaut analysis](http://xkcd.com/893/). Unfortunately, I was unable to find a computationally efficient way to develop an analytical solution, so I fell back to the trusty Monte Carlo simulation.  From that, however, I was quickly able to develop "die-off" curves at 5%, 50% and 95% probabilities. I also added a "birth" curve to describe my Facebook friends coming into the world.

![The living footprint of my Facebook friends](/images/wimsy_friends_lifetime_windows.png "My friends coming into the world, then leaving it at 5%, 50% and 95% probability.")

So, we came into the world beginning in 1937 (with a huge pickup in the 70's). And we'll all (probably) be gone by 2091. Not a bad run...

If you want to try it yourself, head on over to [Actuarize](http://wmsy.me/actuarize), but please be patient. This is a low-volume web app.
