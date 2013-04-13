---
layout: post
title: Was March 2013 Really So Cold?
published: true
tags: [python, pandas, weather, api]
comments: true
---

# Was March Really So Cold?
Wow, this winter seemed long, didn't it? That really hit home in March when the cold weather just seemed to go on and on. It seemed like every day the temperature "felt like it was in the 20's (Fahrenheit). In NYC, in particular, it seemed like - even when the temperature rose to a balmy low 40's, the wind just *would not* give up.

Well, I just discovered that the makers of the excellent [Dark Sky app](http://darkskyapp.com/) have released a new weather app called [forecast.io](http://forecast.io) and that it has an [API](https://developer.forecast.io/) with not only the current forecast, but "current" and "forecast" data going back up to **60 years** in some places.

So, rather than keep speculating about whether this March was really so cold, I can find out for sure - and learn some new data analysis tricks while I'm at it.

### Get everything set up
I'm leveraging a [code snippet](https://gist.github.com/jefftriplett/5260606) I found to handle the forecast.io API, with some minor modifications. Also, I stored some basic data in a config file (initially to protect my forecast.io API key, but also to make it more modular).

<div class="highlight"><pre><span class="kn">from</span> <span class="nn">requests_forecast</span> <span class="kn">import</span> <span class="n">Forecast</span>
<span class="kn">from</span> <span class="nn">config</span> <span class="kn">import</span> <span class="o">*</span>

<span class="n">forecast</span> <span class="o">=</span> <span class="n">Forecast</span><span class="p">(</span><span class="n">FORECAST_API_KEY</span><span class="p">)</span>
<span class="n">forecast</span><span class="o">.</span><span class="n">timezone</span> <span class="o">=</span> <span class="n">LOC_TIMEZONE</span>
</pre></div>



### Select the dates
Initially, I'm choosing to pull weather at noon each day of March for each of the years 2003 - 2013 (the Marches I have lived in New York). If I were really good, the parameters for this would be in the config file too.

<div class="highlight"><pre><span class="kn">from</span> <span class="nn">datetime</span> <span class="kn">import</span> <span class="n">datetime</span>
<span class="n">dates</span> <span class="o">=</span> <span class="p">[]</span>
<span class="n">years</span> <span class="o">=</span> <span class="nb">range</span><span class="p">(</span><span class="mi">2003</span><span class="p">,</span> <span class="mi">2014</span><span class="p">)</span>
<span class="k">for</span> <span class="n">year</span> <span class="ow">in</span> <span class="n">years</span><span class="p">:</span>
    <span class="n">days</span> <span class="o">=</span> <span class="nb">range</span><span class="p">(</span><span class="mi">1</span><span class="p">,</span><span class="mi">32</span><span class="p">)</span>
    <span class="k">for</span> <span class="n">day</span> <span class="ow">in</span> <span class="n">days</span><span class="p">:</span>
        <span class="n">dates</span><span class="o">.</span><span class="n">append</span><span class="p">(</span><span class="n">datetime</span><span class="p">(</span><span class="n">year</span><span class="o">=</span><span class="n">year</span><span class="p">,</span> <span class="n">month</span><span class="o">=</span><span class="mi">3</span><span class="p">,</span> <span class="n">day</span><span class="o">=</span><span class="n">day</span><span class="p">,</span> <span class="n">hour</span><span class="o">=</span><span class="mi">12</span><span class="p">))</span>
</pre></div>



### Download the data
I'm downloading the data here and storing serializing it with [pickle](http://docs.python.org/2/library/pickle.html) so I don't need to keep using API calls after the first successful pull.

<div class="highlight"><pre><span class="kn">import</span> <span class="nn">pickle</span> <span class="kn">as</span> <span class="nn">pkl</span>
<span class="n">forecasts</span> <span class="o">=</span> <span class="p">[</span><span class="n">forecast</span><span class="o">.</span><span class="n">get</span><span class="p">(</span><span class="n">LOC_LAT</span><span class="p">,</span> <span class="n">LOC_LONG</span><span class="p">,</span> <span class="n">time</span><span class="o">=</span><span class="n">date</span><span class="p">)</span> <span class="k">for</span> <span class="n">date</span> <span class="ow">in</span> <span class="n">dates</span><span class="p">]</span>
<span class="n">pkl</span><span class="o">.</span><span class="n">dump</span><span class="p">(</span><span class="n">forecasts</span><span class="p">,</span> <span class="nb">open</span><span class="p">(</span><span class="s">&#39;data/forecasts.pkl&#39;</span><span class="p">,</span> <span class="s">&#39;w&#39;</span><span class="p">))</span>
</pre></div>



### Collect temperatures and wind speeds in a Pandas DataFrame
I'm using [pandas](http://pandas.pydata.org/) for data analysis, natch. For now, I'm focusing solely on temperature and wind speed. I still have the other data in that pickle file should I think up something else I want to see.

<div class="highlight"><pre><span class="kn">from</span> <span class="nn">pandas</span> <span class="kn">import</span> <span class="n">DataFrame</span><span class="p">,</span> <span class="n">DatetimeIndex</span>
<span class="n">weather_df</span> <span class="o">=</span> <span class="n">DataFrame</span><span class="p">({</span><span class="s">&#39;Temperature&#39;</span><span class="p">:</span> <span class="p">[</span><span class="n">x</span><span class="p">[</span><span class="s">&#39;currently&#39;</span><span class="p">][</span><span class="s">&#39;temperature&#39;</span><span class="p">]</span> <span class="k">for</span> <span class="n">x</span> <span class="ow">in</span> <span class="n">forecasts</span><span class="p">],</span>
                        <span class="s">&#39;Wind Speed&#39;</span><span class="p">:</span>   <span class="p">[</span><span class="n">x</span><span class="p">[</span><span class="s">&#39;currently&#39;</span><span class="p">][</span><span class="s">&#39;windSpeed&#39;</span><span class="p">]</span> <span class="k">for</span> <span class="n">x</span> <span class="ow">in</span> <span class="n">forecasts</span><span class="p">]},</span>
                       <span class="n">index</span><span class="o">=</span><span class="n">DatetimeIndex</span><span class="p">([</span><span class="n">x</span><span class="p">[</span><span class="s">&#39;currently&#39;</span><span class="p">][</span><span class="s">&#39;time&#39;</span><span class="p">]</span> <span class="k">for</span> <span class="n">x</span> <span class="ow">in</span> <span class="n">forecasts</span><span class="p">]))</span>
</pre></div>



### Add a wind chill index
I'm using a wind chill formula I found on [Wikipedia](http://en.wikipedia.org/wiki/Wind_chill). 

<div class="highlight"><pre><span class="k">def</span> <span class="nf">windchill</span><span class="p">(</span><span class="n">temp</span><span class="p">,</span> <span class="n">wind</span><span class="p">):</span>
    <span class="k">if</span> <span class="n">temp</span><span class="o">&gt;</span><span class="mi">50</span> <span class="ow">or</span> <span class="n">wind</span><span class="o">&lt;=</span><span class="mi">3</span><span class="p">:</span>
        <span class="k">return</span> <span class="n">temp</span>
    <span class="k">else</span><span class="p">:</span>
        <span class="k">return</span> <span class="mf">35.74</span> <span class="o">+</span> <span class="mf">0.6215</span><span class="o">*</span><span class="n">temp</span> <span class="o">-</span> <span class="mf">35.75</span><span class="o">*</span><span class="n">wind</span><span class="o">**</span><span class="mf">0.16</span> <span class="o">+</span> <span class="mf">0.4275</span><span class="o">*</span><span class="n">temp</span><span class="o">*</span><span class="n">wind</span><span class="o">**</span><span class="mf">0.16</span>

<span class="n">weather_df</span><span class="p">[</span><span class="s">&#39;Wind Chill&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">weather_df</span><span class="o">.</span><span class="n">apply</span><span class="p">(</span>
        <span class="k">lambda</span> <span class="n">row</span><span class="p">:</span> <span class="n">windchill</span><span class="p">(</span><span class="n">row</span><span class="p">[</span><span class="s">&#39;Temperature&#39;</span><span class="p">],</span> <span class="n">row</span><span class="p">[</span><span class="s">&#39;Wind Speed&#39;</span><span class="p">]),</span>
        <span class="n">axis</span> <span class="o">=</span> <span class="mi">1</span><span class="p">)</span>
</pre></div>



### Group by years and take a look at averages

<div class="highlight"><pre><span class="n">weather_df</span><span class="o">.</span><span class="n">groupby</span><span class="p">(</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="n">x</span><span class="o">.</span><span class="n">year</span><span class="p">)</span><span class="o">.</span><span class="n">aggregate</span><span class="p">(</span><span class="n">np</span><span class="o">.</span><span class="n">mean</span><span class="p">)</span><span class="o">.</span><span class="n">plot</span><span class="p">();</span>
</pre></div>



![](/images/March_2013_Weather_Analysis_fig_00.png)


OK, so it looks like the trend in temps and wind chill is *up* if anything. 

<div class="highlight"><pre><span class="n">weather_df</span><span class="o">.</span><span class="n">boxplot</span><span class="p">(</span><span class="n">column</span><span class="o">=</span><span class="s">&#39;Temperature&#39;</span><span class="p">,</span><span class="n">by</span><span class="o">=</span><span class="p">(</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="n">x</span><span class="o">.</span><span class="n">year</span><span class="p">));</span>
</pre></div>



![](/images/March_2013_Weather_Analysis_fig_01.png)


I *love* box plots. It looks like temps stayed in a tight range this March compared to most past Marches. Maybe it was the lack of a single 60+ degree day that drove me batty.

<div class="highlight"><pre><span class="n">weather_df</span><span class="o">.</span><span class="n">boxplot</span><span class="p">(</span><span class="n">column</span><span class="o">=</span><span class="s">&#39;Wind Speed&#39;</span><span class="p">,</span><span class="n">by</span><span class="o">=</span><span class="p">(</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="n">x</span><span class="o">.</span><span class="n">year</span><span class="p">));</span>
</pre></div>



![](/images/March_2013_Weather_Analysis_fig_02.png)


Winds were no higher, on average than recent years.

<div class="highlight"><pre><span class="n">weather_df</span><span class="o">.</span><span class="n">boxplot</span><span class="p">(</span><span class="n">column</span><span class="o">=</span><span class="s">&#39;Wind Chill&#39;</span><span class="p">,</span><span class="n">by</span><span class="o">=</span><span class="p">(</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="n">x</span><span class="o">.</span><span class="n">year</span><span class="p">));</span>
</pre></div>



![](/images/March_2013_Weather_Analysis_fig_03.png)


And, in fact, taking wind and temperature together suggests that - while last year was relatively warm, this year didn't *feel* especially cold compared to prior years.

### Initial conclusion
Based on noon-time measurements alone, it's not obvious that this March should have been any more uncomfortable than past Marches. Possibly a daily average (pulled, say, hourly) would be more instructive.

### Using more data
I decided to pull hourly data from 6AM to 6PM for each of the days to see if a full-day view might yield more info. I wrote a quick Python tool to download more data and store it in Pickle files. I won't go into it here, but I'll put the code on GitHub. Then I pulled the data into a DataFrame called <code>bigdf</code>. 

<div class="highlight"><pre><span class="n">bigdf</span><span class="o">.</span><span class="n">describe</span><span class="p">()</span>
</pre></div>


<pre>
           Temperature   Wind Speed   Wind Chill
    count  4433.000000  4433.000000  4433.000000
    mean     43.541081    11.017018    38.487087
    std       9.928982     4.955918    13.199414
    min      13.320000     0.000000    -6.070190
    25%      37.190000     7.350000    29.845367
    50%      43.270000    10.600000    37.693510
    75%      49.550000    14.100000    46.533495
    max      74.180000    32.390000    74.180000
</pre>


<div class="highlight"><pre><span class="n">bigdf</span><span class="o">.</span><span class="n">groupby</span><span class="p">(</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="n">x</span><span class="o">.</span><span class="n">year</span><span class="p">)</span><span class="o">.</span><span class="n">aggregate</span><span class="p">(</span><span class="n">np</span><span class="o">.</span><span class="n">mean</span><span class="p">)</span><span class="o">.</span><span class="n">plot</span><span class="p">();</span>
</pre></div>



![](/images/March_2013_Weather_Analysis_fig_04.png)


Nothing new here...

<div class="highlight"><pre><span class="n">bigdf</span><span class="o">.</span><span class="n">boxplot</span><span class="p">(</span><span class="n">column</span><span class="o">=</span><span class="s">&#39;Temperature&#39;</span><span class="p">,</span><span class="n">by</span><span class="o">=</span><span class="p">(</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="n">x</span><span class="o">.</span><span class="n">year</span><span class="p">));</span>
<span class="n">bigdf</span><span class="o">.</span><span class="n">boxplot</span><span class="p">(</span><span class="n">column</span><span class="o">=</span><span class="s">&#39;Wind Speed&#39;</span><span class="p">,</span><span class="n">by</span><span class="o">=</span><span class="p">(</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="n">x</span><span class="o">.</span><span class="n">year</span><span class="p">));</span>
<span class="n">bigdf</span><span class="o">.</span><span class="n">boxplot</span><span class="p">(</span><span class="n">column</span><span class="o">=</span><span class="s">&#39;Wind Chill&#39;</span><span class="p">,</span><span class="n">by</span><span class="o">=</span><span class="p">(</span><span class="k">lambda</span> <span class="n">x</span><span class="p">:</span> <span class="n">x</span><span class="o">.</span><span class="n">year</span><span class="p">));</span>
</pre></div>



![](/images/March_2013_Weather_Analysis_fig_05.png)


![](/images/March_2013_Weather_Analysis_fig_06.png)


![](/images/March_2013_Weather_Analysis_fig_07.png)


Rather than look at all the temperatures across the month, let's look at each day's average temperature.

<div class="highlight"><pre><span class="n">avgdf</span> <span class="o">=</span> <span class="n">bigdf</span><span class="o">.</span><span class="n">groupby</span><span class="p">([</span><span class="s">&#39;Year&#39;</span><span class="p">,</span> <span class="s">&#39;Day&#39;</span><span class="p">])</span><span class="o">.</span><span class="n">agg</span><span class="p">(</span><span class="n">np</span><span class="o">.</span><span class="n">mean</span><span class="p">)</span><span class="o">.</span><span class="n">reset_index</span><span class="p">()</span>
<span class="n">avgdf</span><span class="o">.</span><span class="n">boxplot</span><span class="p">(</span><span class="s">&#39;Temperature&#39;</span><span class="p">,</span> <span class="s">&#39;Year&#39;</span><span class="p">);</span>
<span class="n">avgdf</span><span class="o">.</span><span class="n">boxplot</span><span class="p">(</span><span class="s">&#39;Wind Speed&#39;</span><span class="p">,</span> <span class="s">&#39;Year&#39;</span><span class="p">);</span>
<span class="n">avgdf</span><span class="o">.</span><span class="n">boxplot</span><span class="p">(</span><span class="s">&#39;Wind Chill&#39;</span><span class="p">,</span> <span class="s">&#39;Year&#39;</span><span class="p">);</span>
</pre></div>



![](/images/March_2013_Weather_Analysis_fig_08.png)


![](/images/March_2013_Weather_Analysis_fig_09.png)


![](/images/March_2013_Weather_Analysis_fig_10.png)


Well, I got nothin'. Except that, accounting for wind, *this* March didn't seem to have a single daytime hour that felt warmer than 60 degrees - and **that** is legitimately rare for March. It's also arguably in line with the notion that spring was slow in coming this year. In 2011 most temperatures were actually lower, but there were quite a few warm spikes. I don;t know if these were all on one day, since I didn't aggregate the data daily, but it still looks like there was more relief then than this year. The other takeaway? Looking at some of this historic data, it's actually been much worse frequently. 

### Other explanations

So why did March just seem so damned cold? Maybe it was just that the cold lingered late in the month, or that a cold March came after a cold February and we were just sick of it. Maybe understanding the cold requires a more nuanced analysis. Maybe we just like to complain and recent memory is exaggerated in our minds. Obviously my ability to objectively compare successive Marches in my mind over 10 years is, well, suspect.

I **did** get to learn more than I wanted to know about Pandas copies versus views and grouping. Actually I needed to learn this stuff. Hopefully I'll remember it. I also got my first [Stack Overflow post](http://stackoverflow.com/questions/15972264/why-doesnt-this-function-take-after-i-iterrows-over-a-pandas-dataframe) out of the deal.

