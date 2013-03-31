---
layout: post
title: Deep Dive on One Year of Weight Data - Part II
published: true
tags: [python, pandas, quantified self, health]
comments: true
---
# Deep Dive on One Year of Weight Data - Part II

In the [last post](/2013/01/26/deep-dive-on-one-year-of-weight-data---part-i.html), I dug into a year's worth of weight data, describing the patterns and trends I saw, and offering some explanations for each. Here we're going to measure results against the theoretical weight loss based on well-established calorie-to-fat (and muscle) formulas to see just how predictive my recorded food and exercise data really was. 

##Analyzing calories in and calories out
In addition to weighing myself daily, I recorded what I ate and all my exercise using an iPhone app called [LoseIt!](http://loseit.com/). I'll attempt to detere how much of my weight loss was directly attributable to my calorie deficit.

###Calculating calorie deficit
First of all, how does one calculate a calorie deficit?  Generally, it looks like this:
<pre><code>cal_deficit = base_cal_burn + exercise_cal_burn - cal_consumption</code></pre>
where <code>base_cal_burn</code> is based on something called your [Basal Metabolic Rate](http://en.wikipedia.org/wiki/Basal_metabolic_rate), which can be adjusted for "base" activity level, and declines as you lose weight, yielding a total daily energy expenditure (TDEE).

<div class="highlight"><pre><span class="n">base_cal_burn</span> <span class="o">=</span> <span class="n">tdee</span><span class="p">(</span><span class="n">w1y</span><span class="p">,</span><span class="n">height</span><span class="o">=</span><span class="mi">72</span><span class="p">,</span><span class="n">age</span><span class="o">=</span><span class="mi">36</span><span class="p">,</span><span class="n">sex</span><span class="o">=</span><span class="s">&#39;m&#39;</span><span class="p">,</span><span class="n">multiplier</span><span class="o">=</span><span class="mf">1.2</span><span class="p">)</span>
<span class="n">ax</span> <span class="o">=</span> <span class="n">base_cal_burn</span><span class="o">.</span><span class="n">plot</span><span class="p">()</span>
<span class="n">ax</span><span class="o">.</span><span class="n">set_ylabel</span><span class="p">(</span><span class="s">&#39;TDEE - Sedentary (cals)&#39;</span><span class="p">);</span>
</pre></div>



![](/images/Weight_Analysis___First_Pass_fig_05.png)


As you can see, my base calorie burn declined significantly as I lost weight. This is one of the insidious challenges in weight loss.  For a significant change, once you reach your goal you must maintain a much lower calorie intake (or offsetting exercise) to stay there.

<div class="highlight"><pre><span class="n">exercise_cal_burn</span> <span class="o">=</span> <span class="n">Series</span><span class="p">(</span>
        <span class="n">pd</span><span class="o">.</span><span class="n">read_csv</span><span class="p">(</span><span class="s">&#39;data/ExerciseCalories4392.csv&#39;</span><span class="p">,</span> <span class="n">index_col</span><span class="o">=</span><span class="s">&#39;Date&#39;</span><span class="p">,</span>
        <span class="n">parse_dates</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span><span class="o">.</span><span class="n">sort_index</span><span class="p">()[</span><span class="s">&#39;Exercise Calories&#39;</span><span class="p">][</span><span class="s">&#39;2009-06-26&#39;</span><span class="p">:</span><span class="s">&#39;2010-06-22&#39;</span><span class="p">])</span>
<span class="n">ax</span> <span class="o">=</span> <span class="n">exercise_cal_burn</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">style</span><span class="o">=</span><span class="s">&#39;b.&#39;</span><span class="p">)</span>
<span class="n">ax</span><span class="o">.</span><span class="n">set_ylabel</span><span class="p">(</span><span class="s">&#39;Exercise Calories (cal)&#39;</span><span class="p">);</span>
</pre></div>



![](/images/Weight_Analysis___First_Pass_fig_06.png)


As you can see, I exercised regularly throughout my weight loss, with the total energy expenditure increasing signficantly in 2010 as I started running longer distances.

<div class="highlight"><pre><span class="n">cal_consumption</span> <span class="o">=</span> <span class="n">Series</span><span class="p">(</span>
        <span class="n">pd</span><span class="o">.</span><span class="n">read_csv</span><span class="p">(</span><span class="s">&#39;data/FoodCalories4392.csv&#39;</span><span class="p">,</span> <span class="n">index_col</span><span class="o">=</span><span class="s">&#39;Date&#39;</span><span class="p">,</span>
        <span class="n">parse_dates</span><span class="o">=</span><span class="bp">True</span><span class="p">)</span><span class="o">.</span><span class="n">sort_index</span><span class="p">()[</span><span class="s">&#39;Food Calories&#39;</span><span class="p">][</span><span class="s">&#39;2009-06-26&#39;</span><span class="p">:</span><span class="s">&#39;2010-06-22&#39;</span><span class="p">])</span>
<span class="n">ax</span> <span class="o">=</span> <span class="n">cal_consumption</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">style</span><span class="o">=</span><span class="s">&#39;b.&#39;</span><span class="p">)</span>
<span class="n">ax</span><span class="o">.</span><span class="n">set_ylabel</span><span class="p">(</span><span class="s">&#39;Calories Consumed (cal)&#39;</span><span class="p">);</span>
</pre></div>



![](/images/Weight_Analysis___First_Pass_fig_07.png)


It looks like my calorie consumption range was pretty tight earlier on and varied much more widely toward the end of the weight loss period. Also, you'll note some very low and "zero" calorie consumption days. As an example, on August 15, 2009 I apparently consumed only 285 calories. Here's what it looks like:

![](/images/loseit-2009-08-15.jpg)

So, apparently I ate a granola bar (typical pre-run snack), went for a three-mile run, ate a banana (typical post-run snack), then ate nothing else the rest of the day. I suppose this is theoretically possible. I'm more inclined to think I got lazy and stopped recording calories.

So, I looked through my calendar and emails from that week. It turns out I was at a family reunion in upstate New York that weekend. I assure you I ate - and drank - plenty that day and the rest of the weekend. So, this is one of a few data points that I need to clean up. In fact, I think I'll ignore all days that are below a certain threshold on calorie consumption, and assume I had a net calorie deficit of zero. I suspect this is an optimistic assumption, but more on that later.

<div class="highlight"><pre><span class="n">cal_consumption</span><span class="o">.</span><span class="n">hist</span><span class="p">(</span><span class="n">color</span><span class="o">=</span><span class="s">&#39;k&#39;</span><span class="p">,</span> <span class="n">alpha</span><span class="o">=</span><span class="mf">0.5</span><span class="p">,</span> <span class="n">bins</span><span class="o">=</span><span class="mi">20</span><span class="p">);</span>
</pre></div>



![](/images/Weight_Analysis___First_Pass_fig_08.png)


I can believe I had days of calorie consumption in the low 1000's, but anything under 1000 calories is suspect, so I'll mark those (including the zero days) as NULL values.

<div class="highlight"><pre><span class="n">cal_consumption</span> <span class="o">=</span> <span class="n">cal_consumption</span><span class="o">.</span><span class="n">where</span><span class="p">(</span><span class="n">cal_consumption</span> <span class="o">&gt;=</span> <span class="mi">1000</span><span class="p">)</span>
<span class="n">cal_consumption</span><span class="o">.</span><span class="n">hist</span><span class="p">(</span><span class="n">color</span><span class="o">=</span><span class="s">&#39;k&#39;</span><span class="p">,</span> <span class="n">alpha</span><span class="o">=</span><span class="mf">0.5</span><span class="p">,</span> <span class="n">bins</span><span class="o">=</span><span class="mi">20</span><span class="p">);</span>
</pre></div>



![](/images/Weight_Analysis___First_Pass_fig_09.png)


That filtered out about 30 points. Those low ones still look a bit suspect, but let's run with it like this. Those high ones look like outliers, too, but I can't imagine recording calories I didn't actually consume, so I think those are real. I like KDE plots, so let's look at it like that, too.

<div class="highlight"><pre><span class="n">cal_consumption</span><span class="p">[</span><span class="n">cal_consumption</span><span class="o">.</span><span class="n">notnull</span><span class="p">()]</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">kind</span><span class="o">=</span><span class="s">&#39;kde&#39;</span><span class="p">);</span>
</pre></div>



![](/images/Weight_Analysis___First_Pass_fig_10.png)


OK, let's combine it all in a data frame.

<div class="highlight"><pre><span class="kn">from</span> <span class="nn">pandas</span> <span class="kn">import</span> <span class="n">Timestamp</span> <span class="k">as</span> <span class="n">ts</span>
<span class="n">weightdf</span> <span class="o">=</span> <span class="n">DataFrame</span><span class="p">({</span><span class="s">&#39;Weight&#39;</span><span class="p">:</span> <span class="n">w1y</span><span class="p">,</span> <span class="s">&#39;Base Calories&#39;</span><span class="p">:</span> <span class="n">base_cal_burn</span><span class="p">,</span>
                       <span class="s">&#39;Exercise Calories&#39;</span><span class="p">:</span> <span class="n">exercise_cal_burn</span><span class="p">,</span>
                       <span class="s">&#39;Food Calories&#39;</span><span class="p">:</span> <span class="n">cal_consumption</span><span class="p">})</span>
<span class="c"># Drop partial weeks</span>
<span class="n">drop_dates</span> <span class="o">=</span> <span class="p">(</span><span class="s">&#39;2009-6-26&#39;</span><span class="p">,</span> <span class="s">&#39;2009-6-27&#39;</span><span class="p">,</span> <span class="s">&#39;2009-6-28&#39;</span><span class="p">,</span> <span class="s">&#39;2010-6-21&#39;</span><span class="p">,</span> <span class="s">&#39;2010-6-22&#39;</span><span class="p">)</span>
<span class="k">for</span> <span class="n">d</span> <span class="ow">in</span> <span class="n">drop_dates</span><span class="p">:</span>
    <span class="n">weightdf</span> <span class="o">=</span> <span class="n">weightdf</span><span class="o">.</span><span class="n">drop</span><span class="p">(</span><span class="n">ts</span><span class="p">(</span><span class="n">d</span><span class="p">))</span>

<span class="c"># Calculate calorie deficit</span>
<span class="n">weightdf</span><span class="p">[</span><span class="s">&#39;Calorie Deficit&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">weightdf</span><span class="p">[</span><span class="s">&#39;Base Calories&#39;</span><span class="p">]</span> <span class="o">+</span> <span class="n">weightdf</span><span class="p">[</span><span class="s">&#39;Exercise Calories&#39;</span><span class="p">]</span> <span class="o">-</span> <span class="n">weightdf</span><span class="p">[</span><span class="s">&#39;Food Calories&#39;</span><span class="p">]</span>
<span class="k">print</span> <span class="p">(</span><span class="n">weightdf</span><span class="p">[</span><span class="s">&#39;Weight&#39;</span><span class="p">][</span><span class="n">ts</span><span class="p">(</span><span class="s">&#39;6-29-2009&#39;</span><span class="p">)]</span> <span class="o">-</span> <span class="n">weightdf</span><span class="p">[</span><span class="s">&#39;Weight&#39;</span><span class="p">][</span><span class="n">ts</span><span class="p">(</span><span class="s">&#39;6-20-2010&#39;</span><span class="p">)])</span><span class="o">*</span><span class="mi">3500</span><span class="o">/</span><span class="mi">356</span>
<span class="n">weightdf</span><span class="o">.</span><span class="n">describe</span><span class="p">()</span>
</pre></div>


    698.033707865
    

<pre>
           Base Calories  Exercise Calories  Food Calories      Weight  Calorie Deficit
    count     357.000000         357.000000     319.000000  357.000000       319.000000
    mean     2616.102449         228.925728    2269.690721  232.448739       595.182921
    std       135.540163         308.787380     735.925260   18.176104       779.746546
          2405.450116           0.000000    1028.350000  204.200000     -2191.817612
    25%      2496.426157           0.000000    1723.000000  216.400000        82.876077
    50%      2608.281944           0.000000    2043.630000  231.400000       810.317858
    75%      2720.137731         492.029000    2717.205000  246.400000      1146.106112
    max      2942.357895        1485.970000    5329.970000  276.200000      2277.650299
</pre>


On average, I

- burned 2,616 + 229 = 2,845 calories per day.
- consumed 2,270 calories per day.
- had a calorie deficit of 595 calories per day.

Also, I lost 71 pounds over the 356 day period, or an average of 0.20 pounds per day. At 3,500 calories per pound, that equates to 700 calories per day. So, it looks like I lost more weight than I should have given the reported calorie data.

The [next post](/2013/03/31/deep-dive-on-one-year-of-weight-data---part-iii.html) will dig into possible sources of error and explore whether the data themselves can help us understand where I went wrong.
