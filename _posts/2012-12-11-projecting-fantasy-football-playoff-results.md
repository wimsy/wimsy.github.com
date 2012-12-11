---
layout: post
title: Projecting Fantasy Football Playoff Results with Monte Carlo Simulation
published: true
tags: [yql, monte carlo, simulation, python, api, yahoo]
comments: true
---

# Projecting Fantasy Football Playoff Results with Monte Carlo Simulation

For seven or eight years now, I've been involved in one or two fantasy football leagues. For me, the most interesting facet of the game has been the wealth of data available and novel ways to apply it. Team stats, individual player stats, projections, all manner of scoring, drafts, trades, benching vs playing - all of these give a data nerd enough to dissect for days on end. 

So, as I decided to embrace my inner quant, one of the first things I thought I could turn my eye to is one of my fantasy football leagues. This is probably due, in no small part, to my performance this year in one of my leagues.  Whereas I usually find myself at the bottom of the pile, I somehow managed to finish the "regular season" this year ranked #1 in the league, and a guaranteed playoff berth.  Most years at this time, I've pretty much checked out.  Not this time.

I'll talk a little about the tools I used, the process I followed and the results - along with the many weaknesses in this admittedly simplistic approach.

## What I learned
- Using the [Yahoo! Fantasy API](http://developer.yahoo.com/fantasysports/), more specifically the YQL version with the [Python YQL library](http://docs.python-yql.org/en/latest/)
- Deeper understanding of Python and [Pandas](http://pandas.pydata.org/) data structures
- Use of the Python [Pickle](http://docs.python.org/2/library/pickle.html) library for data serialization
- More than I ever wanted to know about 3-legged [OAuth](http://en.wikipedia.org/wiki/OAuth)

Rather than go the standard analytical route, I performed my first simulation - in this case, a simple [Monte Carlo](http://en.wikipedia.org/wiki/Monte_Carlo_method) simulation based on historical scores from the league.  This allowed me to project some (admittedly suspect and impossible to test) probabilities of different outcomes.  I tested the assumptions driving this simulation, showed that they're not completely valid, then totally plowed ahead anyway.

My code, such as it is, is [here](https://github.com/wimsy/ffball).

## How I pulled it together
I originally started down the path of trying to use Yahoo!'s standard [web service](http://developer.yahoo.com/fantasysports/guide/) by forming URLs and parsing the response as I've done before with other APIs. However, I had a great deal of difficulty with this, eventually giving up when authentication broke every time I had to use a semicolon in a query, which is absolutely essential.  A little searching suggests that this is a common issue, but I can't say I exhaustively sought solutions.

After that, I turned to the YQL library from the nice folks over at [Project Fondue](http://projectfondue.com/), and all was revealed! Once I hacked by way through [YQL syntax](http://developer.yahoo.com/yql/) and figured out how it applied to the Fantasy Sports API, I was quickly up and running - downloading our league's scores and stats and playing with the numbers in [iPython](http://ipython.org/).

My initial plan was to do some basic data analysis on the scores and put together some hypotheses from that (time series analysis, forecasting, etc.). But, then it occurred to me that projecting playoff results with a simple Monte Carlo simulation might be more fun (and achievable) in a reasonable amount of time.

## The results
Starting with an initial ranking (top four teams go to the two-round playoffs), I set up matchups for the semifinals, then simulated games based on the historical scoring statistics for each team. Of course, this is incredibly simplistic, but I'm just getting started here!

Then I wrote a Python script to play them over and over, advancing the winning and losing teams to their respective finals positions, and determining the teams' final ranks.  Over 100,000 simulated playoffs, I was able to converge on a probability of each of four contenders to win it all - or achieve any particular final ranking.

Here are the teams' final rankings probabilities:

<table>
	<tr>
		<td>Final Rank</td>
		<td>Ain't no NCAA</td>
		<td>Sproles Royce</td>
		<td>C-Men</td>
		<td>Usual Suspects</td>
	</tr>
	<tr>
		<td>1st</td>
		<td>30%</td>
		<td>30%</td>
		<td>20%</td>
		<td>20%</td>
	</tr>
	<tr>
		<td>2nd</td>
		<td>27%</td>
		<td>27%</td>
		<td>23%</td>
		<td>23%</td>
	</tr>
	<tr>
		<td>3rd</td>
		<td>23%</td>
		<td>23%</td>
		<td>27%</td>
		<td>26%</td>
	</tr>
	<tr>
		<td>4th</td>
		<td>20%</td>
		<td>20%</td>
		<td>30%</td>
		<td>31%</td>
	</tr>
</table>

Of course, I wanted to understand whether my assumption that season-long scoring patterns were indicative of teams' fitness today.  Teams change over time as players are injured, or turn out to be better or worse than expected. Also, coaches' weekly decisions to add or drop players, or play one player over another, can really impact performance.

Implicit in this model is the assumption that scoring is (more or less) normally distributed, and the distribution doesn't change over time. One way to get a sense for that is to take a look at the distributions of scoring.

![Distribution of scores](/images/ffball_dist_1.png "Full-season distribution of scores for the top four teams.")

As you can see, not every scoring history is a clean normal distribution.  The most obvious is that of the *Usual Suspects*.  This appears to be essentially a bimodal distribution where (more often) the team hovers around unremarkable sub-150 point scores, but has a significant number of games in the more impressive high 100's.

My own team, *Ain't no NCAA*, shows a tendency to frequently underperform relative to the more prominent solid performance centered around 160 or so points.  *Sproles Royce* and *C-Men* are more normally distributed and centered at just over 150 points per game.

To get a better feel for how things are looking now (versus a full-season look), I performed the same analysis on just the last seven weeks of the regular season. Warning: Now we're talking about a very small data set, but the graphs are pretty anyway...

![Distribution of scores - last 7 weeks](/images/ffball_dist_2.png "Second-half distribution of scores for the top four teams.")

As you can see, this new look [doesn't bode well for me or my good friends the *Usual Suspects* paints a slightly different picture.

*Ain't no NCAA*: My boys have seemed to either really bring it or stink it up over the last seven weeks, with peaks near 140 and 180.  This makes for a very unpredictable outcome for me.  I could probably model around this bimodal distribution but, hey, it's just a game!

*Sproles Royce*: A more symmetrical distribution I can't hope to see. These guys can expect a consistent playoffs performance if the last seven weeks are indicative, with some slight tendency for outlying good and bad performances.

*C-Men*: These guys have recently been consistently centered around a slightly sub-par level, but with significant departures to the extreme upside.  In normal mode, they can expect to lose at least one, but they have enough upside to bring it home.

*Usual Suspects*: Their recent performances seem to be slightly weighted toward the negative, but they also have some long-tail high-scoring potential.

## Revised simulation
I re-ran the simulation and, as expected, the projected results don't look great for *Ain't no NCAA* or the *Usual Suspects*.

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
		<td>26%</td>
		<td>30%</td>
		<td>27%</td>
		<td>17%</td>
	</tr>
	<tr>
		<td>2nd</td>
		<td>30%</td>
		<td>22%</td>
		<td>21%</td>
		<td>27%</td>
	</tr>
	<tr>
		<td>3rd</td>
		<td>20%</td>
		<td>20%</td>
		<td>29%</td>
		<td>22%</td>
	</tr>
	<tr>
		<td>4th</td>
		<td>24%</td>
		<td>20%</td>
		<td>23%</td>
		<td>34%</td>
	</tr>
</table>

Somewhat ironically, the recent *C-Men* victory that pushed them from 4th to 3rd place has damaged their prospects if recent performance is an indicator of playoffs performance. The two recently strongest teams are in positions 2 and 3, lessening each of their prospects to advance to the finals.  Both are favorites f they do advance, however. 

My projection for the final ranking:
<table>
	<tr>
		<td>Projected Final Rank</td>
		<td>Team</td>
	</tr>
	<tr>
		<td>1st Place</td>
		<td>Sproles Royce</td>
	</tr>
	<tr>
		<td>2nd Place</td>
		<td>Ain't no NCAA'</td>
	</tr>
	<tr>
		<td>3rd Place</td>
		<td>C-Men</td>
	</tr>
	<tr>
		<td>4th Place</td>
		<td>Usual Suspects</td>
	</tr>
</table>

Of course, there's no telling if even this is a valid model. The sample size is small, and there is likely more unutilized predictive power in the data. Maybe the model would be more robust if I'd modeled individual players' performances - certainly it would help to iron out the noise of changing rosters.  Or, how good are the projections provided by Yahoo!? If I had normalized on those would this model be better? Probably.

Who knows? All I really know is that I'm in the playoffs for the first time since I joined this league.  That will do.