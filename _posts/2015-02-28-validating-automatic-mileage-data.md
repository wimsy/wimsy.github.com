---
title: Validating Automatic Mileage & Cost Data
author: "Michael Wimsatt"
date: "February 21, 2015"
output: html_document
layout: post
published: true
tags: [R, mileage, automatic]
comments: true
---

# Validating Automatic data

*Note: Code for this analysis available [on GitHub](https://github.com/wimsy/mileage_analysis/blob/master/validation.Rmd).*

Last March I purchased [Automatic](http://www.automatic.com/), an adapter that plugs into my car's diagnostic port and syncs with an app on my iPhone to report and record data about my driving. Among other things, Automatic provides:

- route, miles and time driven
- gas consumed
- estimated cost of gas consumed
- gas-wasting events like hard braking and speeds over 70 mph

During the first few months, I had trouble getting the adapter to kick in reliably when a trip started, and it still alerts me regularly that it hasn't connected to the smartphone app. Reliability has certainly improved, but I wanted to get an idea of accuracy -- is the cost and gas mileage being reported actually close to what I'm experiencing? To get some independent data, I started recording odometer readings, gallons and price at each fill-up in August.

So... How well do the two datasets match up?



## The data

I have two datasets. One downloaded from the Automatic Dashboard. Each row is a trip. As mentioned above, there are missing trips in this dataset. 


{% highlight text %}
## 'data.frame':	696 obs. of  13 variables:
##  $ Vehicle                    : chr  "2008 Volkswagen Jetta" "2008 Volkswagen Jetta" "2008 Volkswagen Jetta" "2008 Volkswagen Jetta" ...
##  $ Start.Time                 : POSIXct, format: "2014-03-06 07:59:00" "2014-03-06 11:47:00" ...
##  $ End.Time                   : POSIXct, format: "2014-03-06 09:27:00" "2014-03-06 11:59:00" ...
##  $ Distance..mi.              : num  52.77 6.4 6.51 0.51 42.02 ...
##  $ Duration..min.             : num  88.08 12.32 12.69 5.07 63.73 ...
##  $ Fuel.Cost..USD.            : num  7.69 1.04 0.95 0.2 5.48 5.46 0.78 6.27 0.29 1.64 ...
##  $ Average.MPG                : num  25.68 22.79 25.45 9.57 28.64 ...
##  $ Fuel.Volume..gal.          : num  2.05 0.28 0.26 0.05 1.47 1.46 0.21 1.68 0.08 0.43 ...
##  $ Hard.Accelerations         : int  0 0 0 1 0 0 0 0 0 0 ...
##  $ Hard.Brakes                : int  3 1 0 0 1 1 0 4 0 0 ...
##  $ Duration.Over.70.mph..secs.: int  39 0 7 0 6 17 0 0 0 0 ...
##  $ Duration.Over.75.mph..secs.: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ Duration.Over.80.mph..secs.: int  0 0 0 0 0 0 0 0 0 0 ...
{% endhighlight %}

The second is a set of data recorded at each fill-up. 


{% highlight text %}
##              Timestamp Gallons Price  Miles diff.miles
## 43 2015-01-21 16:22:48   13.31 28.47 150291        364
## 44 2015-01-28 16:57:25   11.96 30.60 150619        328
## 45 2015-02-03 08:55:15   11.90 29.04 150918        299
## 46 2015-02-06 17:51:12   11.55 29.10 151233        315
## 47 2015-02-13 07:48:49   13.71 35.62 151564        331
## 48 2015-02-20 17:25:46   11.92 32.89 151900        336
{% endhighlight %}

## Overall accuracy

Generally, Automatic appears to be accurate when it actually registers a trip. 



| Measure           | Automatic Data | Mileage Data |
| ------------------|---------------:| ------------:|
| Miles driven      | 8,893 | 9,348 |
| Gallons consumed  | 326 | 345 |
| Fuel cost | $ 1,011 | $ 1,095 |
| Miles per gallon | 27.3 | 27.1 |
| Price per gallon | $ 3.105 | $ 3.174 |

Here you can see that, while total mileage and fuel consumption are off, the price per gallon and gas mileage are pretty close to actuals.

## Monthly analysis

Even over time, the Automatic rate data seems accurate. The total monthly mileage tracks well between actuals and Automatic's calculations. I suspect the major month-to-month differences are a result of missed trips and timing mismatches between the trips data and my fill-up mileage data.

![plot of chunk comparison](/images/2015-02-28-validating-automatic-mileage-data/comparison.png) 

Rate data (mpg and cost per gallon) show even better accuracy.

![plot of chunk mpg](/images/2015-02-28-validating-automatic-mileage-data/mpg.png) 

![plot of chunk cpg](/images/2015-02-28-validating-automatic-mileage-data/cpg.png) 

It's a little disappointing to see the once-plummeting gasoline price level off in February, but it had to end eventually...

## Getting a little more precise

Since trips and fill-ups don't line up perfectly with months, I'd like to compare the trips data directly with each tank of gas. This also gives me a chance to play with time intervals in the [lubridate](http://cran.r-project.org/web/packages/lubridate/index.html) package.



![plot of chunk tankdf](/images/2015-02-28-validating-automatic-mileage-data/tankdf.png) 

*The big swing you see at the end of November in my mileage records is the result of stopping at a gas station that had some technical problems and I was able to fill my tank only partially. Since accurate estimates of fuel consumption require me to fill the tank each time, this skewed the numbers for this fill-up and the following one.*

![plot of chunk cpg2](/images/2015-02-28-validating-automatic-mileage-data/cpg2.png) 

![plot of chunk miles](/images/2015-02-28-validating-automatic-mileage-data/miles.png) 

## Conclusion

As you can see in the graphs above, cost per gallon, gas mileage and even total miles driven tracked pretty well with my manually recorded data -- the exceptions coming mostly when AUtomatic missed trips. There are probably other systematic sources of error, too. For instance, I usually pay a slightly higher credit price at the pump, and Automatic might be assuming the lower cash price.
