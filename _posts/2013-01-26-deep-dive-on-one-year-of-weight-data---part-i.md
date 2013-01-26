---
layout: post
title: Deep Dive on One Year of Weight Data - Part I
published: true
tags: [python, pandas, quantified self, health]
---

# Deep Dive on One Year of Weight Data - Part I

In 2009, I finally decided to get serious about losing some weight. I was near my all-time peak of 290 pounds and feeling terrible. I was also spending my weeks near work several hours from home, and nothing but free time on my hands. Rather than spend it watching tv and eating junk food, I though I'd try to prepare healthy meals and give running a shot.

Over the following year I managed to lose 75 pounds, developed a running habit, including running races up to 15K, and generally learned to appreciate a healthy lifestyle. This isn't the whole story (lowlight - I gained about half back; highlight: I ran a marathon), but more on that in future posts.

This is about the data! This post and succeeding ones will be about my trying to learn some exploratory data analysis skills using [iPython Notebook](http://ipython.org/ipython-doc/dev/interactive/htmlnotebook.html) (what an incredible tool!), [pandas](http://pandas.pydata.org/), [matplotlib](http://matplotlib.org/), [SciPy](http://www.scipy.org/), and other tools.

During the course of my weight loss from mid-2009 to mid-2010, I weighed myself nearly every day. Additionally, I recorded food and exercise calories most days. Let's look at the results.

*Note: I have some separate scripts that you will see me using here. You can see my code (plus the original iPython Notebook) [on GitHub](https://github.com/wimsy/weight).*

<div class="highlight"><pre><span class="kn">import</span> <span class="nn">matplotlib.pyplot</span> <span class="kn">as</span> <span class="nn">plt</span>
<span class="n">run</span> <span class="n">weight_analysis</span><span class="o">.</span><span class="n">py</span>
<span class="kn">from</span> <span class="nn">weight_utils</span> <span class="kn">import</span> <span class="o">*</span>
</pre></div>

## On why you should weigh yourself daily
When you're losing weight, they tell you to weigh yourself weekly (or less often), because the journey downward can be, well, indirect:

<div class="highlight"><pre><span class="n">ax</span> <span class="o">=</span> <span class="n">w1y</span><span class="o">.</span><span class="n">plot</span><span class="p">()</span>
<span class="n">ax</span><span class="o">.</span><span class="n">set_ylabel</span><span class="p">(</span><span class="s">&#39;Weight (lbs)&#39;</span><span class="p">);</span>
</pre></div>



![](/images/Weight_Analysis___First_Pass_fig_00.png)


But there are ways to deal with the noise of daily weight fluctuation. A method that I found helpful was that outlined in [The Hacker's Diet](http://www.fourmilab.ch/hackdiet/). Namely, use a moving average of your weight as your indicator of progress. Since it's a lagging indicator, if you are losing weight most of your weights measurements will be *below* average, bringing the average down and giving you more "losing weight" days. I like how Erv Walter demonstrates it graphically [here](http://www.ewal.net/2012/10/05/day-to-day-weight-fluctuations-and-mental-stress/). 

<div class="highlight"><pre><span class="n">ax</span> <span class="o">=</span> <span class="n">w1y</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">style</span><span class="o">=</span><span class="s">&#39;b.&#39;</span><span class="p">,</span> <span class="n">alpha</span><span class="o">=</span><span class="mf">0.2</span><span class="p">)</span>
<span class="n">smooth</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">style</span><span class="o">=</span><span class="s">&#39;r&#39;</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">title</span><span class="p">(</span><span class="s">&#39;Daily Weight Data (Smoothed and Unsmoothed)&#39;</span><span class="p">)</span>
<span class="n">ax</span><span class="o">.</span><span class="n">set_ylabel</span><span class="p">(</span><span class="s">&#39;Weight (lbs)&#39;</span><span class="p">);</span>
</pre></div>



![](/images/Weight_Analysis___First_Pass_fig_01.png)


### But why not just weight yourself weekly? Isn't that also "smoothing"?
Well sure, you could do that, but then you'd lose out on all this wonderful **data**! For instance, don't you want to know while you feel pretty bloated after a football weekend with your college pals?  Well, maybe it's because you **gained ten pounds** which you promptly lost with barely a blip in the overall trend.

<div class="highlight"><pre><span class="n">ax</span> <span class="o">=</span> <span class="n">w1y</span><span class="p">[</span><span class="s">&#39;09-2009&#39;</span><span class="p">]</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">title</span><span class="o">=</span><span class="s">&#39;Daily Weight Data - September 2009&#39;</span><span class="p">)</span>
<span class="n">smooth</span><span class="p">[</span><span class="s">&#39;09-2009&#39;</span><span class="p">]</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">style</span><span class="o">=</span><span class="s">&#39;r&#39;</span><span class="p">)</span>
<span class="n">ax</span><span class="o">.</span><span class="n">set_ylabel</span><span class="p">(</span><span class="s">&#39;Weight (lbs)&#39;</span><span class="p">);</span>
</pre></div>



![](/images/Weight_Analysis___First_Pass_fig_02.png)

More on this later, but I found that having daily data gave me real insight into how my body reacts to changes in diet and activity, further helping me to react calmly and thoughtfully to inevitable weight fluctuations. It also gave me an intuitive understanding for why impermanent weight-loss solutions are so impermanent.

##Measuring weight loss rate
You can also estimate your short- and long-term weight loss rate. And, as usual, using daily data makes this infinitely more fun! I just calculated the day-to-day weight difference, converted it to a weekly rate, and smoothed it with a 7-day and 90-day window (the former giving me more instantaneous, but noisy, insight - and the latter describing the overall trend).

<div class="highlight"><pre><span class="n">ax</span> <span class="o">=</span> <span class="n">pd</span><span class="o">.</span><span class="n">ewma</span><span class="p">(</span><span class="n">smooth</span><span class="o">.</span><span class="n">diff</span><span class="p">()</span><span class="o">*-</span><span class="mi">7</span><span class="p">,</span> <span class="mi">90</span><span class="p">)</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">title</span><span class="o">=</span><span class="s">&#39;Weight Loss Rate in Pounds per Week&#39;</span><span class="p">,</span> <span class="n">label</span><span class="o">=</span><span class="s">&#39;90-day window&#39;</span><span class="p">)</span>
<span class="n">pd</span><span class="o">.</span><span class="n">ewma</span><span class="p">(</span><span class="n">smooth</span><span class="o">.</span><span class="n">diff</span><span class="p">()</span><span class="o">*-</span><span class="mi">7</span><span class="p">,</span> <span class="mi">7</span><span class="p">)</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">color</span><span class="o">=</span><span class="s">&#39;r&#39;</span><span class="p">,</span> <span class="n">label</span><span class="o">=</span><span class="s">&#39;7-day window&#39;</span><span class="p">)</span>
<span class="n">ax</span><span class="o">.</span><span class="n">set_ylabel</span><span class="p">(</span><span class="s">&#39;Weekly Weight Loss (negative numbers are Gains)&#39;</span><span class="p">)</span>
<span class="n">ax</span><span class="o">.</span><span class="n">legend</span><span class="p">();</span>
</pre></div>


![](/images/Weight_Analysis___First_Pass_fig_03.png)

As you can see, I started off losing a lot of weight quickly (pretty common, especially if you're coming off a period of near-constant fried food and beer). My rate of weight loss steadily declined through August and September, until it settled in at about two pound per week (although September included the aforementioned football weekend blip). At the holidays, my weight loss dropped off to a little over a pound per week (my budget was for 1.5 pounds - I wasn't perfect), but you can see the individual holidays (Thanksgiving and Christmas/New Years) where I actually gained weight briefly. I still don;t know what happened in early February. Speculation on that later. My weight loss was pretty steady through the spring, then settled in at about a pound per week to finish it off.

##Weight fluctuation patterns
One pretty predictable element of my weight loss fluctuations was the weekly pattern I observed along the way. Since for the most part, my weight loss efforts were more focused during the week, a pretty consistent pattern emerged. I would typically see dramatic weight loss Wednesday through Saturday followed by dramatic weight gains Sunday through Tuesday. Once I understood these patterns I could shrug off the upticks early in the week and look forward to the plummeting measurements at the end, always feeling pretty good going into the weekend.

<div class="highlight"><pre><span class="n">daily_pattern</span> <span class="o">=</span> <span class="n">np</span><span class="o">.</span><span class="n">concatenate</span><span class="p">((</span><span class="n">np</span><span class="o">.</span><span class="n">zeros</span><span class="p">(</span><span class="mi">1</span><span class="p">),</span> <span class="n">np</span><span class="o">.</span><span class="n">array</span><span class="p">(</span><span class="n">avg_dds</span><span class="p">[</span><span class="s">&#39;Padded&#39;</span><span class="p">])))</span><span class="o">.</span><span class="n">cumsum</span><span class="p">()</span>
<span class="n">fig</span><span class="p">,</span> <span class="n">ax</span> <span class="o">=</span> <span class="n">plt</span><span class="o">.</span><span class="n">subplots</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>
<span class="n">ax</span><span class="o">.</span><span class="n">plot</span><span class="p">(</span><span class="n">daily_pattern</span><span class="p">)</span>
<span class="n">locs</span><span class="p">,</span> <span class="n">labels</span> <span class="o">=</span> <span class="n">xticks</span><span class="p">()</span>
<span class="n">xticks</span><span class="p">(</span><span class="n">np</span><span class="o">.</span><span class="n">arange</span><span class="p">(</span><span class="mi">8</span><span class="p">),</span> <span class="p">(</span><span class="s">&#39;Sun&#39;</span><span class="p">,</span> <span class="s">&#39;Mon&#39;</span><span class="p">,</span> <span class="s">&#39;Tue&#39;</span><span class="p">,</span> <span class="s">&#39;Wed&#39;</span><span class="p">,</span> <span class="s">&#39;Thu&#39;</span><span class="p">,</span> <span class="s">&#39;Fri&#39;</span><span class="p">,</span> <span class="s">&#39;Sat&#39;</span><span class="p">,</span> <span class="s">&#39;Sun&#39;</span><span class="p">))</span>
<span class="n">ax</span><span class="o">.</span><span class="n">set_ylabel</span><span class="p">(</span><span class="s">&#39;Average weight gain (loss) from prior day&#39;</span><span class="p">)</span>
<span class="n">plt</span><span class="o">.</span><span class="n">show</span><span class="p">()</span>
</pre></div>



![](/images/Weight_Analysis___First_Pass_fig_04.png)

So, that's the weight data itself (the outcome). But, what about the inputs that drove this weight loss - calories in and calories out? That's the next post in this series.

