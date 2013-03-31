---
layout: post
title: Deep Dive on One Year of Weight Data - Part III
published: true
tags: [python, pandas, quantified self, health]
comments: true
---

# Deep Dive on One Year of Weight Data - Part III

In the [last post](/2013/03/01/deep-dive-on-one-year-of-weight-data---part-ii.html), we did a deep dive on calories consumed and burned to test how close actual results were to predicted results. While they were (perhaps surprisingly) close, they weren't right on. I seemed to lose more weight than I should have based on my average calorie deficit over the year. So, why might that be?

## Analyzing sources of error

What are some possible sources of error in this analysis?

- Errors in estimated calorie consumption
- Errors in estimated exercise calories
- Errors in base calorie consumption estimate
- Missing data
- Errors in the 3,500 calories per pound estimate

I would expect the calorie consumption and missing data (basically more calorie consumption) to be skewed the *opposite* direction (overestimating calorie deficit), and I have no reason to question my exercise calories, so if this is a result of miscalculating the calorie deficit it probably comes down to the estimated base calorie consumption.

Of course, this is wrapped up in the estimated exercise calories, since it varies based on average base activity level. My recorded exercise is meant to include just "exercise events" - running, mostly, plus some long walks, and maybe some strength training here and there. But, through the course of the day at work, and on weekends (I live in the very walkable New York City), I'm walking a lot (maybe three miles per day, on average). Some of this walking is included in the "sedentary" multiplier of 1.2, but perhaps not enough?

Let's calculate the change in TDEE for various activity levels based on my average weight and age during the weight loss period:

<div class="highlight"><pre><span class="k">for</span> <span class="n">mult</span> <span class="ow">in</span> <span class="n">arange</span><span class="p">(</span><span class="mf">1.2</span><span class="p">,</span> <span class="mf">1.5</span><span class="p">,</span> <span class="o">.</span><span class="mo">05</span><span class="p">):</span>
    <span class="k">print</span> <span class="nb">str</span><span class="p">(</span><span class="n">mult</span><span class="p">)</span> <span class="o">+</span> <span class="s">&#39;</span><span class="se">\t</span><span class="s">&#39;</span> <span class="o">+</span> <span class="nb">str</span><span class="p">(</span><span class="n">tdee</span><span class="p">(</span><span class="mi">235</span><span class="p">,</span><span class="mi">72</span><span class="p">,</span><span class="mi">36</span><span class="p">,</span><span class="s">&#39;m&#39;</span><span class="p">,</span><span class="n">mult</span><span class="p">)</span> <span class="o">-</span> <span class="n">tdee</span><span class="p">(</span><span class="mi">235</span><span class="p">,</span><span class="mi">72</span><span class="p">,</span><span class="mi">36</span><span class="p">,</span><span class="s">&#39;m&#39;</span><span class="p">,</span><span class="mf">1.2</span><span class="p">))</span>
</pre></div>


    1.2	0.0
    1.25	109.7969722
    1.3	219.5939444
    1.35	329.3909166
    1.4	439.1878888
    1.45	548.984861
    1.5	658.7818332


The first takeaway here is that this calculation is very sensitive to activity level (at least for men of a certain size). When we measure calorie deficits in hundreds of calories per day, the difference between "sedentary" and "lightly active" in our base calorie consumption is almost 400 calories. That means if you are trying to lose one pound per week, choosing the wrong activity level could nearly obliterate (or double) your weight loss! Whoa!

It looks like a multiplier of 1.3 would equate to an extra 220 calories deficit per day. This is more than the 100 calories we need to explain, but I think we have some oppositely skewed errors as mentioned above, and 1.3 seems to be pretty close to where the LoseIt! app is estimating an office worker's activity level.

While this could explain why I seemingly lost more weight than I should have, I'd also like to understand how my imperfect data collection might have skewed results.

### Estimating the missing impact of missing data

So, now I want to check my assumptions about the missing data being skewed toward exaggerating my calorie deficit. One way would be to look for discrepancies at the weekly level (daily weight data is plagued by noise, as illustrated above).

First, to take as much noise out as possible, I'm going to work with the moving average weight as illustrated above.

<div class="highlight"><pre><span class="n">weightdf</span><span class="p">[</span><span class="s">&#39;Weight Trend&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">smooth</span>
<span class="n">weightdf</span><span class="p">[</span><span class="s">&#39;Weight Trend&#39;</span><span class="p">][</span><span class="n">ts</span><span class="p">(</span><span class="s">&#39;6-29-2009&#39;</span><span class="p">)]</span> <span class="o">-</span> <span class="n">weightdf</span><span class="p">[</span><span class="s">&#39;Weight Trend&#39;</span><span class="p">][</span><span class="n">ts</span><span class="p">(</span><span class="s">&#39;6-20-2010&#39;</span><span class="p">)]</span>
</pre></div>


<pre>
    67.804100457827616
</pre>


Note that since the weight trend is a lagging indicator, it shows a slightly lower weight loss (68 lbs vs 71 lbs) than the absolute weight measurements.  This works out to .19 lbs per day and a calorie deficit of 670 calories. These are a bit different from what we saw above, but small compared to the size of the errors we're trying to assess.

Now let's turn our daily data into weekly data.

###Resampling daily data to weekly

First, let's create an indicator which will give us an "average" number of missing data days we have in a week. We'll also use Pandas' resample function to aggregate average weekly data for each week in the set.

<div class="highlight"><pre><span class="n">weightdf</span><span class="p">[</span><span class="s">&#39;Missing Data&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">weightdf</span><span class="p">[</span><span class="s">&#39;Calorie Deficit&#39;</span><span class="p">]</span><span class="o">.</span><span class="n">isnull</span><span class="p">()</span> <span class="o">*</span> <span class="mi">1</span>
<span class="n">wkweightdf</span> <span class="o">=</span> <span class="n">weightdf</span><span class="p">[[</span><span class="s">&#39;Base Calories&#39;</span><span class="p">,</span> <span class="s">&#39;Exercise Calories&#39;</span><span class="p">,</span> 
                       <span class="s">&#39;Food Calories&#39;</span><span class="p">,</span> <span class="s">&#39;Calorie Deficit&#39;</span><span class="p">]]</span><span class="o">.</span><span class="n">resample</span><span class="p">(</span><span class="s">&#39;W-SUN&#39;</span><span class="p">,</span> 
                                                                     <span class="n">how</span><span class="o">=</span><span class="s">&#39;mean&#39;</span><span class="p">,</span> <span class="n">kind</span><span class="o">=</span><span class="s">&#39;period&#39;</span><span class="p">)</span>
<span class="n">wkweightdf</span><span class="p">[</span><span class="s">&#39;Missing Data&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">weightdf</span><span class="p">[[</span><span class="s">&#39;Missing Data&#39;</span><span class="p">]]</span><span class="o">.</span><span class="n">resample</span><span class="p">(</span><span class="s">&#39;W-SUN&#39;</span><span class="p">,</span> 
                                                                 <span class="n">how</span><span class="o">=</span><span class="s">&#39;sum&#39;</span><span class="p">,</span> <span class="n">kind</span><span class="o">=</span><span class="s">&#39;period&#39;</span><span class="p">)</span>
<span class="n">wkweightdf</span><span class="p">[</span><span class="s">&#39;Final Weight&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">weightdf</span><span class="p">[[</span><span class="s">&#39;Weight Trend&#39;</span><span class="p">]]</span><span class="o">.</span><span class="n">resample</span><span class="p">(</span><span class="s">&#39;W-SUN&#39;</span><span class="p">,</span> 
                                                                 <span class="n">how</span><span class="o">=</span><span class="s">&#39;last&#39;</span><span class="p">,</span> <span class="n">kind</span><span class="o">=</span><span class="s">&#39;period&#39;</span><span class="p">)</span>
<span class="n">start_weight</span> <span class="o">=</span> <span class="n">smooth</span><span class="p">[</span><span class="s">&#39;2009-6-28&#39;</span><span class="p">]</span>
<span class="n">wkweightdf</span><span class="p">[</span><span class="s">&#39;Weight Lost&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="o">-</span><span class="n">wkweightdf</span><span class="p">[</span><span class="s">&#39;Final Weight&#39;</span><span class="p">]</span><span class="o">.</span><span class="n">diff</span><span class="p">()</span>
<span class="n">wkweightdf</span><span class="o">.</span><span class="n">describe</span><span class="p">()</span>
</pre></div>


<pre>
           Base Calories  Exercise Calories  Food Calories  Calorie Deficit  Missing Data  Final Weight  Weight Lost
    count      51.000000          51.000000      51.000000        51.000000     51.000000     51.000000    50.000000
    mean     2616.102449         228.925728    2255.773560       605.649044      0.745098    231.886921     1.290055
    std       135.951091          93.386778     229.313245       285.993383      1.180562     18.016530     1.103964
    min      2421.003397          36.115429    1732.755000       -70.308145      0.000000    205.732605    -1.362209
    25%      2496.532686         180.785714    2099.063929       409.795619      0.000000    215.728934     0.550104
    50%      2611.690882         235.428571    2271.230000       574.015235      0.000000    231.019346     1.363960
    75%      2718.113674         291.338571    2405.318571       800.780589      1.500000    244.978118     2.103550
    max      2907.842395         438.086143    2770.237500      1156.036502      5.000000    270.235350     3.374956
</pre>


Ideally, we would have summed some, struck a difference among others, and took averages on yet others, but I got lazy and just took averages. Possible correction in the future.

## Looking at the data

Now, for what you've all been waiting to see - a scatter matrix!

<div class="highlight"><pre><span class="n">pd</span><span class="o">.</span><span class="n">scatter_matrix</span><span class="p">(</span><span class="n">wkweightdf</span><span class="p">[[</span><span class="s">&#39;Base Calories&#39;</span><span class="p">,</span> <span class="s">&#39;Exercise Calories&#39;</span><span class="p">,</span> <span class="s">&#39;Food Calories&#39;</span><span class="p">,</span> <span class="s">&#39;Calorie Deficit&#39;</span><span class="p">,</span>
                             <span class="s">&#39;Missing Data&#39;</span><span class="p">,</span> <span class="s">&#39;Weight Lost&#39;</span><span class="p">]],</span> <span class="n">figsize</span><span class="o">=</span><span class="p">(</span><span class="mi">10</span><span class="p">,</span><span class="mi">10</span><span class="p">),</span> <span class="n">diagonal</span><span class="o">=</span><span class="s">&#39;kde&#39;</span><span class="p">);</span>
</pre></div>



![](/images/Weight_Analysis___First_Pass_fig_11.png)


What jumps out here? 

* There does seem to be a bit of correlation between number of days of missing data and (lack of) weight loss, or at least the very worst weight weeks were also those where I recorded data on only a few days. 
* There's the expected positive correlation between calorie deficit and weight loss. 
* There's an apparently strong correlation between base calories and calorie deficit (i.e. the higher my base calorie burn, the higher my calorie deficit).
* There might be a positive correlation between exercise calories and food calories, but that's to be expected because I was managing toward a net calorie budget.
* There's no apparent strong correlation between exercise calories and calorie deficit, but food calories are strongly negatively correlated with calorie deficit.

Let's dig into a few of these observations a little more closely.

### Missing data

While there does appear to be some relationship between missing data and weight loss (mostly in the extreme), it doesn't appear to be incredibly strong.  But it does show that what we don't know is probably skewing total estimated calorie deficit to the optimistic side. In general, though, missing a couple of days in a week doesn't seem to throw off the numbers too badly. I believe the value of tracking plays out more in longer term trends (as evidenced by the weight gain I experienced at the end of this process when I stopped tracking altogether).

### Weight Loss vs Calorie Deficit

The correlation is positive, as we'd expect, which seems to imply that the weight loss is experienced somewhat coincident with the behavior change (or the delay is within a week's time). However, it may be that a week with good behavior is more likely to be surrounded by weeks of good behavior and the effects seen in one well-behaved week might be the result of the week or weeks before it. It's still satisfying to see the relationship here, though. 

As to the specific relationship (i.e. does the weight loss match theory?), I think that deserves some more targeted analysis. We'll come back to this.

### Base Calories as driver of Calorie Deficit

Since the calorie deficit is a calculation involving base calories, it's not surprising to see a relationship here. And it is clear that as my weight dropped and my base calorie consumption fell, I did not maintain the same calorie deficit. This plays out in the graph of my weight loss. Toward the end I was losing about one pound per week as opposed to the 1.5+ pounds I was losing early on. I simply was not cutting my food calories (or adding offsetting exercise) at a rate sufficient to maintain my target calorie deficit.

I don't know if maintaining that deficit was just less realistic at the lower weight or if, after six months of pretty strict calorie control, I just started letting go a bit. I know my lifestyle began to loosen up about that time, so I suspect it's a mix of the two. 

I guess this isn't really correlation analysis of independent phenomena since one is calculated from the other, but it gets at which pieces of the equation contribute the most variance.

### Exercise vs Food as drivers of weight loss

I'm sure you've read many times that exercise is not sufficient for weight loss. That was definitely my experience. My exercise appears to have no correlation with calorie deficit or weight loss. It was all about the food I ate and my base calorie burn. Food was obviously a much more sensitive lever on my calorie deficit. Absent ultramarathon training, it's just very hard to get enough exercise calories to offset a diet out of control.

Exercise is certainly important for overall health, and running is an addition to my lifestyle that I will value as long as I can still get out there. It also made the food restrictions more bearable. On days when I ran, I could maybe have an extra beer or two without torpedoing my goals. Also, I feel (but don't have the data to show) that the general healthfulness that comes with running also subtly encourages me to make healthier food choices.

## How predictive is my Calorie Deficit model?

Now let's take a closer look at the calorie deficit and related weight loss. Is "calories in and calories out" a sufficient model for weight loss? Let's model what the theoretical weight loss was for each week and compare it to the results.

### Calculating theoretical weekly weight loss

First we need to estimate how much weight i *should* have lost in a given week. For a few reasons I won't go into here, I'll go back to the daily data, then recalculate the weekly data.

<div class="highlight"><pre><span class="n">wkweightdf</span><span class="p">[</span><span class="s">&#39;Total Deficit&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">weightdf</span><span class="p">[[</span><span class="s">&#39;Calorie Deficit&#39;</span><span class="p">]]</span><span class="o">.</span><span class="n">resample</span><span class="p">(</span><span class="s">&#39;W-SUN&#39;</span><span class="p">,</span> 
                                                                 <span class="n">how</span><span class="o">=</span><span class="s">&#39;sum&#39;</span><span class="p">,</span> <span class="n">kind</span><span class="o">=</span><span class="s">&#39;period&#39;</span><span class="p">)</span>
<span class="n">wkweightdf</span><span class="p">[</span><span class="s">&#39;Theoretical Weight Loss&#39;</span><span class="p">]</span> <span class="o">=</span> <span class="n">wkweightdf</span><span class="p">[</span><span class="s">&#39;Total Deficit&#39;</span><span class="p">]</span><span class="o">/</span><span class="mf">3500.0</span>
<span class="n">plt</span><span class="o">.</span><span class="n">figure</span><span class="p">(</span><span class="n">figsize</span><span class="o">=</span><span class="p">(</span><span class="mi">5</span><span class="p">,</span><span class="mi">5</span><span class="p">))</span>
<span class="n">plt</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">wkweightdf</span><span class="p">[</span><span class="s">&#39;Theoretical Weight Loss&#39;</span><span class="p">],</span> <span class="n">wkweightdf</span><span class="p">[</span><span class="s">&#39;Weight Lost&#39;</span><span class="p">],</span> <span class="s">&#39;bo&#39;</span><span class="p">);</span>
<span class="n">plt</span><span class="o">.</span><span class="n">ylabel</span><span class="p">(</span><span class="s">&#39;Actual Weight Lost (lb)&#39;</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">xlabel</span><span class="p">(</span><span class="s">&#39;Projected Weight Lost (lb)&#39;</span><span class="p">);</span>
<span class="n">z</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">polyfit</span><span class="p">(</span><span class="n">wkweightdf</span><span class="p">[</span><span class="s">&#39;Theoretical Weight Loss&#39;</span><span class="p">]</span><span class="o">.</span><span class="n">tail</span><span class="p">(</span><span class="mi">50</span><span class="p">),</span><span class="n">wkweightdf</span><span class="p">[</span><span class="s">&#39;Weight Lost&#39;</span><span class="p">]</span><span class="o">.</span><span class="n">tail</span><span class="p">(</span><span class="mi">50</span><span class="p">),</span><span class="mi">1</span><span class="p">)</span>
<span class="n">m</span> <span class="o">=</span> <span class="n">z</span><span class="p">[</span><span class="mi">0</span><span class="p">];</span> <span class="n">b</span> <span class="o">=</span> <span class="n">z</span><span class="p">[</span><span class="mi">1</span><span class="p">]</span>
<span class="n">x</span> <span class="o">=</span> <span class="n">linspace</span><span class="p">(</span><span class="o">-</span><span class="mi">2</span><span class="p">,</span><span class="mi">4</span><span class="p">)</span>
<span class="n">y</span> <span class="o">=</span> <span class="n">m</span><span class="o">*</span><span class="n">x</span> <span class="o">+</span> <span class="n">b</span>
<span class="n">plt</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">x</span><span class="p">,</span><span class="n">y</span><span class="p">,</span><span class="s">&#39;r-&#39;</span><span class="p">,</span> <span class="n">label</span><span class="o">=</span><span class="s">&#39;Fit&#39;</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">x</span><span class="p">,</span><span class="n">x</span><span class="p">,</span><span class="s">&#39;r--&#39;</span><span class="p">,</span> <span class="n">label</span><span class="o">=</span><span class="s">&#39;Perfect&#39;</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">xlim</span><span class="p">(</span><span class="o">-</span><span class="mi">2</span><span class="p">,</span><span class="mi">4</span><span class="p">);</span>
<span class="n">plt</span><span class="o">.</span><span class="n">ylim</span><span class="p">(</span><span class="o">-</span><span class="mi">2</span><span class="p">,</span><span class="mi">4</span><span class="p">);</span>
<span class="n">plt</span><span class="o">.</span><span class="n">legend</span><span class="p">(</span><span class="n">loc</span><span class="o">=</span><span class="s">&#39;best&#39;</span><span class="p">);</span>
</pre></div>



![](/images/Weight_Analysis___First_Pass_fig_12.png)


From this data, it appears that my actual weight loss experience was more sensitive to calorie deficit than the standard rule of 3,500 calories per pound would imply. Why might that be?

1. There aren't 3,500 calories per pound.
2. It's a little more complex than 3,500 calories per pound if body fat/muscle composition are taken into account.
3. There were biases in my estimation of calories (in and out) that seemed to be proportionally off to the same degree (or the data would look less linear)

So, I did a little research and number 2 seems to be the culprit. While there are 3,500 metabolizable calories in a pound of fat, there are only 600 metabolizable calories in a pound of muscle. With a slope of 1.35 pounds per pound between actual and theoretical, it looks like I was burning, on average, 2,500 calories per pound lost. I'll leave the math to you, but this works out to me losing about one pound of muscle for every pound of fat I lost over the year.

<div class="highlight"><pre><span class="n">bfpct</span> <span class="o">=</span> <span class="n">fatwatch_data</span><span class="p">[</span><span class="s">&#39;Body Fat&#39;</span><span class="p">][:</span><span class="s">&#39;06-2010&#39;</span><span class="p">]</span> <span class="c"># Some recorded body fat % data in original file</span>
<span class="c"># bfpct[bfpct==0] = NaN                         # Drop zero values in sparse data</span>
<span class="n">fat</span> <span class="o">=</span> <span class="p">(</span><span class="n">bfpct</span> <span class="o">*</span> <span class="n">w1y</span><span class="o">/</span><span class="mi">100</span><span class="p">)</span>                         <span class="c"># body fat</span>
<span class="n">muscle</span> <span class="o">=</span> <span class="n">w1y</span> <span class="o">-</span> <span class="n">fat</span>                            <span class="c"># muscle</span>
<span class="n">bodycomp</span> <span class="o">=</span> <span class="n">DataFrame</span><span class="p">({</span><span class="s">&#39;Fat&#39;</span><span class="p">:</span> <span class="n">fat</span><span class="p">,</span> <span class="s">&#39;Muscle&#39;</span><span class="p">:</span> <span class="n">muscle</span><span class="p">,</span> <span class="s">&#39;Body Fat %&#39;</span><span class="p">:</span> <span class="n">bfpct</span><span class="p">})</span>
<span class="k">for</span> <span class="n">col</span> <span class="ow">in</span> <span class="n">bodycomp</span><span class="o">.</span><span class="n">columns</span><span class="p">:</span>
    <span class="n">bodycomp</span><span class="p">[</span><span class="n">col</span><span class="p">]</span> <span class="o">=</span> <span class="n">bodycomp</span><span class="p">[</span><span class="n">col</span><span class="p">]</span><span class="o">.</span><span class="n">interpolate</span><span class="p">()</span>
<span class="n">bodycomp</span><span class="p">[:</span><span class="s">&#39;05-2010&#39;</span><span class="p">]</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">secondary_y</span><span class="o">=</span><span class="p">[</span><span class="s">&#39;Body Fat %&#39;</span><span class="p">],</span> <span class="n">style</span><span class="o">=</span><span class="s">&#39;-&#39;</span><span class="p">);</span>
</pre></div>



![](/images/Weight_Analysis___First_Pass_fig_13.png)


Well, this chart seems to belie my earlier conclusion. If these body fat percentages are to be believed, I managed to gain muscle mass during the year of weight loss. To be honest, I'm skeptical of this. My body fat measurements came from a standard body weight scale - which is notoriously inaccurate. I was doing almost no strength training during this period, and it's hard to imagine that running alone would have made up the difference. I'll admit, I'm at a loss. Any thoughts?

# Conclusion

In this series of posts, we used tools from the Python data analysis ecosystem to really dig into the data I generated while losing weight over one year. While there were no earth-shattering discoveries, I got more familiar with some excellent tools (pandas, iPython notebook), and some minor insight into what is important and not so important in weight loss and maintenance.

I'm back on the weight-loss wagon, and now I'm tracking a lot more data. I use a FitBit One to track motion for calorie consumption. I'm much more diligent about capturing calorie consumption. I'm tracking my sleep nightly, too. Hopefully, I'll get down to my goal and have another success story to dissect in a year or so.

