---
layout: post
title: Projecting Fantasy Football Playoff Results with Monte Carlo Simulation
published: true
category: Learning
tags: [yql, monte carlo, simulation, python, api, yahoo]
---

# Projecting Fantasy Football Playoff Results with Monte Carlo Simulation

For me, the most interesting facet of fantasy football has been the wealth of data available and novl ways to apply it. Fantasy baseball fans might argue that their game has even more of it but, honestly, who has the time?

So, as I decided to embrace my inner quant, one of the first things I thought I could turn my eye to is one of my fantasy football leagues. This turned out to be an auspicious year to get this thought, as (unlike most years) I find myself at the top of my league (soon to be bumped to #2 I fear) and with a guaranteed playoff berth. This keeps me interested enough to give it a go this time around.

## What I learned
- Using the [Yahoo! Fantasy API](http://developer.yahoo.com/fantasysports/), more specifically the YQL version with the [Python YQL library](http://docs.python-yql.org/en/latest/).
- Deeper understanding of Python and [Pandas](http://pandas.pydata.org/) data structures.
- Use of the Python [Pickle](http://docs.python.org/2/library/pickle.html) library for data serialization.
- More than I ever wanted to know about 3-legged [OAuth](http://en.wikipedia.org/wiki/OAuth).

Also, rather than go the standard analytical route, I performed my first simulation - in this case, a simple [Monte Carlo](http://en.wikipedia.org/wiki/Monte_Carlo_method) simulation based on historical scores from the league.

## How I pulled it together
I originally started down the path of trying to use Yahoo!'s standard [web service](http://developer.yahoo.com/fantasysports/guide/) by forming URLs and parsing the response as I've done before with other APIs. However, I had a great deal of difficulty with this, eventually giving up when authentication broke every time I had to use a semicolon in a query, which is absolutely essential.  

So, I turned to the YQL library from the nice folks over at [Project Fondue](http://projectfondue.com/), and all was revealed! Once I learned by way through [YQL syntax](http://developer.yahoo.com/yql/), and figured out how it applied to the Fantasy Sports API, I was quickly up and running, downloading our league's scores and stats, and playing with the numbers.

My initial plan was to do some basic data analysis on the scores and put together some hypotheses from that (trending, standard deviations, etc.). But, then it occurred to me that projecting playoff results with a simple Monte Carlo simulation was the way to go.

## The results
Starting with an initial ranking (top four teams go to the two-round playoffs), I set up matchups for the semifinals, then simulated games based on the historical scoring statistics for each team (of course, this is incredibly simplistic, but I'm just getting started here!).

Then I had my Python script play them over and over, advancing the winning and losing teams to their respective finals positions, and determining the teams' final ranks.  Over 100,000 simulated playoffs, I was able to converge on a probability of each of four contenders to win it all - or achieve any particular final ranking.

Here are the teams' final rankings probabilities:

<table>
	<tr>
		<td>Final Rank</td>
		<td>Ain't no NCAA</td>
		<td>Sproles Royce</td>
		<td>Usual Suspects</td>
		<td>C-Men</td>
	</tr>
	<tr>
		<td>1st</td>
		<td>31%</td>
		<td>32%</td>
		<td>19%</td>
		<td>17%</td>
	</tr>
	<tr>
		<td>2nd</td>
		<td>29%</td>
		<td>26%</td>
		<td>22%</td>
		<td>23%</td>
	</tr>
	<tr>
		<td>3rd</td>
		<td>21%</td>
		<td>24%</td>
		<td>28%</td>
		<td>27%</td>
	</tr>
	<tr>
		<td>4th</td>
		<td>19%</td>
		<td>17%</td>
		<td>30%</td>
		<td>34%</td>
	</tr>
</table>

Of course, I wanted to understand whether my assumption that season-long scoring patterns were indicative of teams' fitness today.  Teams changed over time as players are injured, or turn out to be better or worse than expected, and coaches' weekly decisions to add or drop players, or play one player over another, can really impact performance.

Implicit in this model is the assumption that scoring is (more or less) normally distributed, and the distribution doesn't change over time. One way to get a sense for that is to take a look at the distributions of scoring.

![Distribution of scores](/images/ffball_dist_1.png "Full-season distribution of scores for the top four teams.")

As you can see, not a single one of these is a clean normal distribution.  The most obvious is that of the *Usual Suspects*.  This appears to be essentially a bimodal distribution where (more often) the team hovers around unremarkable sub-150 point scores, but has a significant number of games in the more impressive high 100's.

My own team, *Ain't no NCAA*, shows a tendency to frequently underperform relative to the more prominent solid performance centered around 160 or so points.  *Sproles Royce* is perhaps the most normal, but with a disturbing lack of low-performing games as noted by the trunated left side.  Finally, the *C-Men* have a very consistent average performance, with a few trips into sub-125 territory.

To get a better feel for how things are looking now (versus a full-season look), I performed the same analysis on just the last seven weeks of the regular season.

![Distribution of scores - last 7 weeks](/images/ffball_dist_2.png "Second-half distribution of scores for the top four teams.")

As you can see, this new look doesn't bode well for me or my good friends the *Usual Suspects*. Our second-half performances have been decidedly sub-par, although my team benefited from one or two outlier performances timed perfectly to keep me at the top of the standings.  Meanwhile, *Sproles Royce* has come on strong in the final weeks (even occupying the top spot for one week), and that's obvious with their consistently high scoring.  And the *C-Men* have managed to carry their consistency through the final seven weeks.

## Revised simulation
I re-ran the simulation and, as expected, the projected results are not quite so sangunie for my *Ain't no NCAA* or the *Usual Suspects*.

<table>
	<tr>
		<td>Final Rank</td>
		<td>Ain't no NCAA</td>
		<td>Sproles Royce</td>
		<td>Usual Suspects</td>
		<td>C-Men</td>
	</tr>
	<tr>
		<td>1st</td>
		<td>22%</td>
		<td>38%</td>
		<td>15%</td>
		<td>25%</td>
	</tr>
	<tr>
		<td>2nd</td>
		<td>25%</td>
		<td>27%</td>
		<td>20%</td>
		<td>28%</td>
	</tr>
	<tr>
		<td>3rd</td>
		<td>27%</td>
		<td>20%</td>
		<td>27%</td>
		<td>25%</td>
	</tr>
	<tr>
		<td>4th</td>
		<td>26%</td>
		<td>15%</td>
		<td>38%</td>
		<td>22%</td>
	</tr>
</table>

In fact, the now-4th place *C-Men* have a higher likelihood of winning it all than my *Ain't no NCAA* and *Sproles Royce* - in second place now - is the heavy favorite to win the championship.  

Of course, there's no telling if even this is a valid model. The sample size is small, and there is likely more predictive power in the data available.  Maybe the model would be more robust if I'd modeled indiviudal players' performances - certainly it would help to iron out the noise of changing rosters.  Or, how good are the projections provided by Yahoo!? If I had normalized on those would this model have more predictive power? Probably.

Who knows? All I really know is that I'm in the playoffs for the first time since I joined this league.  That will do.