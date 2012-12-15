---
layout: post
title: Actuarize to Creep Out Your Friends with Actuarial Tables
published: true
tags: [fql, facebook, xkcd, python, api, actuary]
comments: true
---

# Actuarize to Creep Out Your Friends with Actuarial Tables

I don't know. I guess I was feeling a bit morose this week. Nah, actually I've wanted to do this ever since I came across [xkcd](http://www.xkcd.com)'s [clever use of actuarial tables](http://blog.xkcd.com/2012/07/12/a-morbid-python-script/). Also, I wanted an excuse to try out the [Facebook Open Graph API](https://developers.facebook.com/docs/reference/apis/). 

So, I'm going to use cold, hard statistics to tell my friends when they can expect to die. Not any particular friend, mind you, but some statistical sampling of them. I even worked with my friend @ScottLNorvell to whip up a little web app that anyone can do to get the same basic stats that xkcd pulled together.

## What I learned
- I had my first ugly experience with [Facebook's OAuth process](https://developers.facebook.com/docs/howtos/login/login-for-desktop/). I'm not going to say that I exhausted it (or even learned what I was doing wrong) but it was painful nonetheless. I think if I ever get serious about using Facebook data, I'll need to revisit this at length.
- After my failed attempt at Facebook's OAuth, I punted and chose to use [Temboo](http://www.temboo.com) to accomplish the login task. They provide the necessary Callback URL taht I found so challenging the first time around (the idea for a web app came later).
- I used the standard [Facebook Python SDK](@pythonforfacebook/facebook-sdk) with [FWL](https://developers.facebook.com/docs/reference/fql/) for the data requests themselves.
- @ScottLNorvell adapted xkcd's [actuary.py script](http://blog.xkcd.com/2012/07/12/a-morbid-python-script/) to receive input from downloaded Facebook data and return the standard response.
- I took my first run at building something on the [Flask](http://flask.pocoo.org/) web framework, along with mbr/flask-bootstrap for formatting.