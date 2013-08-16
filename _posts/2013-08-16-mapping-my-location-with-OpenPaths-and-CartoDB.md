---
layout: post
published: true
title: Mapping My Location with OpenPaths and CartoDB
tags: [learning, mooc, geography, python, pandas]
comments: true
---

# Mapping My Location with OpenPaths and CartoDB

I'm just wrapping up a Coursera course called [Maps and the Geospatial Revolution](https://www.coursera.org/course/maps). The final project is a map of our choosing, and I thought this would be a great opportunity to finally use the location data I've been collecting through [OpenPaths](http://openpaths.cc). 

I recently became aware of a NYC-based mapping startup called [CartoDB](http://cartodb.com) through an excellent [gallery](http://andrewxhill.github.io/cartodb-examples/scroll-story/pluto/index.html) and decided I'd give that a whirl in place of the also-quite-nice [ArcGIS](http://www.arcgis.com/) tool used in the class. I used Pandas in an iPython notebook for the data preparation, and even had to write some CartoDB-flavored CSS to get the maps looking just right. The iPython notebook and some related code are available [on GitHub](https://github.com/wimsy/maps_project). Finally, I chose my color schemes over at [ColorBrewer](http://colorbrewer2.org).

There was a lot of work upfront, cleaning up the data, developing meaningful timestamps in the correct timezones, classifying data points as weekday or weekend, and calculating distances using latitude and longitude. You can see most of that work (including some false starts and some really ugly code) in the iPython notebook on GitHub.

## Dwell time and weekends vs weekdays

In the first map, I segregated data points into weekends and weekdays, and the bubble size represents the amount of time at a given location. You can [see](http://cdb.io/1d9VOEk) that I have very distinct patterns on weekdays, and different patterns on weekends. My weekdays are spent commuting to and from and spending time at my two offices on Long Island and north of Philadelphia. My weekends are spent in and around my neighborhood in western [Queens](http://www.queenstech.org/) and traveling to central Pennsylvania and upstate New York.

<iframe width='100%' height='400' frameborder='0' src='http://wimsy.cartodb.com/viz/533e739c-0560-11e3-80b6-dfe95a3edaf2/embed_map?title=true&and;description=true&and;search=false&and;shareable=false&and;cartodb_logo=true&and;layer_selector=false&and;legends=true&and;scrollwheel=false&and;sublayer_options=1&and;sql=&and;sw_lat=40.46157664398329&and;sw_lon=-74.63836669921874&and;ne_lat=40.950862628132775&and;ne_lon=-73.32000732421875'> </iframe>

## How far do I roam (and for how long)?

In the [second map](http://cdb.io/13owb0N), I wanted to get a feel for how far afield I am from home for a given percentage of my life. The colors represent successively greater percentages of time, starting at home and measuring outward. 

<iframe width='100%' height='400' frameborder='0' src='http://wimsy.cartodb.com/viz/3bde937c-05ff-11e3-996c-4fa1c4c34f60/embed_map?title=true&and;description=true&and;search=false&and;shareable=false&and;cartodb_logo=true&and;layer_selector=false&and;legends=true&and;scrollwheel=false&and;sublayer_options=1&and;sql=&and;sw_lat=37.405073750176946&and;sw_lon=-85.166015625&and;ne_lat=45.1510532655634&and;ne_lon=-64.072265625'> </iframe>

Here's what I learned:

- So, about a quarter of my time I spend on my own block. Given that I should be sleeping about 1/3 of the time, that seems right (especially when accounting for nights away and some dwell points that didn't update to my home and ended up just outside the 0.1-mile radius).

- The one-mile radius marks what I'd call central Astoria - the neighborhood where I usually live and play. And I spend a little over half my time there.

- I'd say 10 miles is a decent approximation of New York City (sorry Staten Island). I spend about 60% of my time within the [four](http://cityroom.blogs.nytimes.com/2008/12/17/a-new-call-for-staten-island-to-secede/?_r=0) boroughs.

- The 40-mile line encompasses my normal office and about 83% of my life. Expanding to 100 miles includes another office near Philadelphia, which I visit weekly, and most my other regional business travel - bumping me up to 91%.

- If we expand the radius to 200 miles, you catch most of my weekend trips home, upstate or elsewhere in the spring and summer. They add about 4 percentage points to account for 95% of my time. 

- The final 5%? One ten-day trip to Arizona and California in early May. You need to expand the radius to 2,600 miles to pick that one up and to capture all the time I've spent since late January this year.

## Wrapping it up

So, what did I learn, overall? Well, I kind of dig CartoDB. I think their free tier (5 table limit, 5MB limit, 10,000 map views) is designed more as a try-before-you-buy alternative than a free-for-personal-use tool. So, while I had fun, I'm not sure I could keep using it unless I were making maps for a living. I have the sense that it can get very powerful if you're willing to get into the SQL- and CSS-driven capabilities.

OpenPaths (and [Moves](http://moves-app.com)) are great little apps, and I'm happy to finally be realizing the benefit of the battery drain I've been imposing on myself for the last six months or so. I've been periodically downloading the data using their APIs to make sure I have all the current data when I need it. I'm a big fan of owning your own data, and am gradually building scripts to download and keep my cloud data locally.

[I really dig maps!](http://www.amazon.com/Map-Addict-Obsession-Ordnance-Survey/dp/0007351577) None of the results were surprises, but I spent hours panning and zooming on my data - gleaning tiny insights and remembering moments in my life over the last few months. I'm glad I took this course, and I'll be seeking out more opportunities to combine my learning about data analysis and coding with maps to create compelling visualizations and gain new insights.