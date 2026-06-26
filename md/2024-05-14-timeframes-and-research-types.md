# Timeframes & Research Types

Source HTML: [`html/2024-05-14-timeframes-and-research-types.html`](../html/2024-05-14-timeframes-and-research-types.html)

# Timeframes & Research Types

| 항목 | 값 |
| --- | --- |
| 날짜 | 2024-05-14 |
| 접근 | 무료 |
| URL | https://www.algos.org/p/timeframes-and-research-types |
| 부제 | Mechanical vs. Statistical vs. Hypothesis driven research |

---

[![](images/46fc30ed804e.png)](images/d89468e9fee3.png)

#### Introduction

---

This is a departure from my usual article topics and is a lot more abstract. It’s a mental framework for how I think about the different types of research you end up doing—one that is heavily dependent on the timeframe of the strategy itself.

None of what I say below is anything close to a defined truth we must all believe in. It’s a theory of research that I came up with over my time as a quant, and I feel it is an accurate categorization of how strategy research differs. There are blends of the different types and exceptions across the board, but I think this is a helpful generalization of the work. A dimensionality reduction in some ways - pardon my quant talk.

The Quant Stack is a reader-supported publication. To receive new posts and support my work, consider becoming a free or paid subscriber.

#### Clarity

---

So what are the different types of strategy research types, and how do they differ? Broadly, we have:

1. Mechanical
2. Statistical
3. Hypothesis

These are defined by where they sit on the scale of clarity. How clear are the effects we are observing, and how much information does the data hand over to us?

**Mechanical** is a clear cause-and-effect relationship without much doubt in the matter. You’ll need a handful of examples and can come to conclusions with a high amount of conviction to those conclusions.

**Statistical** deals with probabilities and informed guesswork. As an example:

“We could have overfit our backtest… and this could have happened by random chance… but overall, we think this evidence gives good enough clarity to trade it.”

**Hypothesis** deals with fundamental reasoning about a problem and why something may occur. You use this when there simply isn’t enough data to go off alone.

#### Decreasing Accuracy

---

As our timeframe increases, the accuracy of our forecasts decreases [[ref](https://jfin-swufe.springeropen.com/articles/10.1186/s40854-019-0157-x)]. You can get an insanely high R² off a linear regression for an HFT imbalance alpha (not that you will necessarily be able to make money off of it because of mechanical reasons we will get into), but you’ll have a very hard time forecasting the next years return for the S&P500.

That is to say, the longer we look outwards in time, the harder it gets, so we must rely more on the overall instead of individual cases and, eventually, our reasoning as to why something may or may not occur.

#### Sticking to what you know.

---

Most researchers will focus on a rough area of timeframe. They may explore them all, but it is unlikely to be done equally. The firms that trade each timeframe differ significantly and structurally; it doesn’t make much sense for a firm with a structure of:

1. High AUM
2. Low % fee on performance
3. Low average Sharpe

To run a strategy with:

1. Low capacity
2. High Sharpe

Higher Sharpe typically warrants a higher performance fee, so it would usually be run by a fund like this:

1. Low AUM
2. High % fee on performance
3. High average Sharpe

If a firm charges 30% on performance and focuses on higher Sharpe strategies, it will earn 3x as much as a mega fund charging 10% on performance and will outcompete that mega fund in terms of the cost-benefit analysis of allocating research resources to such an alpha.

Compensation structures vary heavily; here are my rough guesses, but of course, people have exceptions, and I don’t know everything:

- Large long-term funds: 0.5-1.5% / 5-15%
- Quite big, but higher Sharpe (think Millennium/ Citadel funds): 1-2% / 20-30%
- Prop pods: Salary or 0 / 20-50%
- (you are a pod) at shops like Millennium: Salary or 0 / 10-15%
- SMAs - seen a lot of 2&20s and 0&30s

As you can see, deals vary all the time, but it all depends on what you are doing, whether you are a pod in a larger firm or the fund itself, how much capital, Sharpe, so many variables. The highest I ever saw was 50% of PNL + 50% of the capital was their own, so an effective 75%, but their fixed pay was negative, and they paid the firm to use their tooling, infra, back-office, etc. Those guys were actual killers, though, and surely got to write their ticket in that case. Despite that skill in their respective area, you wouldn’t expect them to start managing 3bn running CTA momentum, and not just because they don’t know much about it.

These are not “industry standards;” they are my guesses based on the data points in my head.

[![](images/185972818e28.jpeg)](images/d913f5794569.jpeg)Take with a pinch of salt - or more

Sometimes, we see Sharpe-dependent performance fees or tiered performance fees, but generally, it is hard for a low-frequency manager to make it worthwhile to start running low-capacity, high-Sharpe strategies, and as a result, people tend to stick to what they know best. There’s not much hope of this changing either; large investors have an expectation of a reasonably simple fee structure - if you give them a function or table to determine payout, they usually don’t like it. One piece of advice I got from our hedge fund advisor when that was a thing is that institutional investors don’t like any deviation from the norm in fund structures.

This is all to say - get ready to pick a type of research as your favorite.

#### Why this matters?

---

The question of why any of this matters is quite a simple one. If you are going to be spending most of your time mostly doing one type of research, you better understand what they entail and what you enjoy/find you are great at.

Long-term factor strategies tend not to need a complex understanding of advanced machine learning because you don’t really have enough data to do any of it (rare exceptions exist\*; this is not a sweeping statement). It can come down to the very topics a researcher studies before working in the industry - so don’t waste time!

#### Examples of Mechanical Research

---

Arbitrage and market-making strategies are the ones where you will have the most focus on mechanically driven research. You will have a mix of both when looking at HFT statistical alphas as it blends the two categories.

A core characteristic of mechanical research is that individual examples can tell you all you need to know. Mechanical research, on occasion, can involve some statistics, but it is usually things like averages and bar charts. You may decide to plot the average PNL for an arbitrage by the exchange (including the PNL of the overall trade if the exchange is any leg of that arbitrage), and then you could use that to spot that one exchange consistently is a money loser.

You whip out a couple of examples and realize you are consistently too slow on that exchange by about 50 milliseconds.

Giving orderbook imbalances as a statistical HFT alpha, we can see how it blends both. You will want to optimize the alpha to be the best-performing one possible, but at the same time, you must consider how the orderbook actually functions. If you get filled, you are in front of the queue - this means that if you notice an imbalance and place a limit order, by the time you are front of the queue, it means the imbalance has disappeared. So, despite your R² being amazing, you haven’t made a profitable strategy.

Simple plots, basic descriptive statistics, and clear causal effects - often seen from a handful of examples only are core elements of it. At the same time, it is very hard to simulate things, as discussed above, with our amazing R² but poor live performance. You take a lot of feedback from production trading and fundamental design.

#### Examples of Statistical Research

---

As you move up towards medium and lower-frequency research, you end up relying more on statistical performance. What is the R²? How does the simulation look? Sure, simulations can have all sorts of errors, but in terms of simulating how that strategy would have performed if you had known about it at that time (regardless of whether your assumption that knowing about it at the time is correct or not - i.e., overfitting) you know that it will be quite easy to accurately simulate it. If you’ve got borrowed cost data and quote-level data for execution, you can almost exactly simulate the performance of the strategy.

So, it becomes a question of whether your strategy is the issue. You could have gotten lucky, overfitted, introduced biases, etc.

Testing in production is not free—it may be a viable option for some lower timeframe strategies as you can get enough samples to have confidence quickly, but often, a lot of thought and care is taken before you actually trade it because the wait is not a day—it’s months.

You can basically do whatever you want and not worry about overfitting if you are investigating the mechanics of an arbitrage strategy - but for a medium frequency strategy, you have to be extremely strict in terms of not testing too many ideas, sticking to the hypothesis, and ensuring logic makes fundamental sense.

#### Examples of Hypothesis Research

---

This brings us to the topic of hypotheses. When all else gives out, you only have your intuition and reason to rely upon. You’ll develop this over time as you see effects in markets. There are clear patterns, such as impact ripples, that you will see in markets. You’ll get to understand the dance between price, volume, sentiment, shocks, regime, mean-reversion, and momentum eventually.

Everyone has their own theories about why these things happen, and you can, of course, never truly “solve the market” with a completely descriptive theory, but when an alpha makes sense fundamentally and aligns with our reasoning, the probability that it is not a statistical fluke increases drastically.

If we switch our hypothesis against our reasoning every time we see the data contradict it - and continue to test it, then we are no better than a brute force algorithm. What’s the point of an idea in the first place, then? You may as well just code up an alpha search algorithm - these already exist, and that data has been mined to death.

For our hypothesis to have any meaning, we must do as much as we can in terms of design before seeing the data. Then, once we’ve seen the data, we either discard it and work on something entirely new or press ahead, having confirmed our hypothesis.

As you work with markets more and more, you’ll build up an idea of why hypotheses tend to work out, even across different markets, and this is how you end up building this skill out.

We see this mostly with longer-term factor-based strategies. You don’t see a single one of these without an attempt to explain why it happens. Nothing is ever left to being a statistical pattern when you read papers on the longer timeframe effects. Now, a lot of these are done after the effects are found or built off what others have already discovered. We must be pure. We must avoid these pitfalls and not construct our hypothesis after seeing the data.

There is a fine line between tuning your intuition from what you’ve seen and using past work to effectively come up with a hypothesis after seeing the data, but for most researchers, I think by the time you get to this point - you’ve become wise enough to find the balance anyways.

#### Thoughts at the end

---

This was quite a casual article today. I speak with authority throughout the article, but know that there is not any defined way to approach it - I only hope you will use this framework to build your own view of how the research process differs. I’ve been lucky enough to work in medium frequency at the start and then high-frequency now - I still, on occasion have a great medium frequency idea and test it out. Why not, after all? Profit is profit. My longer-term alphas experience is basically at a hobbyist level. I am no expert here, but there are still parallels in terms of coming up with hypotheses for medium-frequency and longer-term effects.

Don’t waste time trying to think about what exact IS or ISN’T medium/high/low frequency. It’s a blurred line based on many different people’s definitions of it. I talked about this question a fair bit in Ep. 1 of Tick Talk if it is that pressing, however (I get this question a lot), but frankly, I don’t think it matters.

For an example of how I’ve developed theories for markets, look at my article on momentum tips/tricks; I discuss a fair bit of my view on the mean-reversion vs. momentum dance with volume. I also spent a few articles giving my own crackpot theory to how pairs trading should be thought of and approached.

Here are the articles:

The correct way to do it, per se. Not “Ken Griffin is manipulating markets” tin foil hat theories, but like this article. A dimensionality reduction into 3 simple areas, throwing away some data but allowing us to actually sit down and think about what we’re doing. Reflecting is improving, as always.

This is also a personal note for myself; that’s how I started on Twitter. I still laugh at my theories of manifolds and neural networks from many, many years back I’d write out on pen & paper, but this one comes from a lot more practical experience, so I hope it’ll last the test of time, and if not it’ll be a fun laugh for future me.

The Quant Stack is a reader-supported publication. To receive new posts and support my work, consider becoming a free or paid subscriber.
