# Beginner Mistakes in Quant

Source HTML: [`html/2025-07-12-beginner-mistakes-in-quant.html`](../html/2025-07-12-beginner-mistakes-in-quant.html)

# Beginner Mistakes in Quant

| 항목 | 값 |
| --- | --- |
| 날짜 | 2025-07-12 |
| 접근 | 유료 |
| URL | https://www.algos.org/p/beginner-mistakes-in-quant |
| 부제 | The most common mistakes we all make early in our quant journey |

---

[![Vintage Sci Fi HD Wallpaper | 1920x1080 | ID:61219 - WallpaperVortex.com](images/6538348c7531.jpeg)](images/be3b0aad5b8c.jpeg)

### Introduction

---

In my time, both working in the industry, and seeing the questions and projects of those in my DMs, I’ve come to see many repeated mistakes and fundamental flaws of thinking made by beginners.

You will see with a very strong degree of commonality that we all seem to make the same mistakes early on in our quant careers. It’s practically guaranteed you will fall for at least a few of these, and I was no different at the start of my career. It’s part of the learning process, but hopefully today I can speed up that process by making explicit where people go wrong. I’m not sure all of these mistakes can be truly avoided, you kind of need to have it beaten out of you (if by no one else, then the market will do it for you).

### Index

---

1. Introduction
2. Index
3. Market Making
4. Machine Learning
5. Models vs. Features
6. Dashboards
7. Time Optimisation
8. Cynicism
9. Nothing beats doing it

### Market Making

---

Let’s start with market making. Market making is one of the areas of literature where the academic material is perhaps the furthest from the work of the practitioners. Perhaps that is because they are attempting to describe an incredibly complex and evolving dynamic system with neat mathematical models in many cases which miss all of the nuance. The issue with the world of HFT is adversity. If your model is wrong, then the areas where you are wrong are when you get filled, so the effect of being wrong compounds quite aggressively.

There is also a fairly large degree of misplaced focus. When it comes to market making you roughly have the below model:

1. You come up with a fair value (usually midprice + some adjustment based on your forecast of where midprice will be in the future)
2. You add a spread on that based on your expectation of risk
3. You skew around that fair value based on your inventory

This is a fairly simple model, of course, and you will often solve much more nuanced sub-problems such as how should I place my orders in front of large uninformed orders to lower the chance of the price moving against me if I get filled (since there’s lots of size to get filled behind you making it hard to move price) OR should you one up another order since it’s quite a large order and you are much less likely to get filled being behind it and can increase fill probability in exchange for tightening your spread a very small amount. These are just to name a couple of them…

But, at a high level, these are your 3 canonical problems. They are also ranked in order of priority. The literature tends to care mostly about skewing which is really not that important of a problem. In reality, what makes money is knowing when to blow out your spreads because a shitstorm is incoming in the book AND having a good idea of fair price (which is basically just having edge in your trading). If you don’t have edge in your trading you won’t go very far. All of the literature tries to create these complex PDEs for skewing risk, when in reality a simple linear skew function will work perfectly fine. In fact, in many cases the optimal function isn’t even a nice equation because there is signalling involved about your inventory when you skew too much (mostly for larger players).

They also like to take the deep reinforcement learning approach; this isn’t so great for a couple of reasons. First of all, not all problems can be specified. For example, one time we were losing money because we had allowed our BTC & ETH correlations to be used to cancel their inventories out against each other (based on the correlations), but this ended up causing huge pickoffs because when we had a large long position on the BTC/ETH spread we would get picked off on where the BTC/ETH spread was going and when we had a large short position the same would happen. Effectively we would get picked off with respect to others predicting the future correlation and we would accumulate adverse inventory at sub-optimal times. These are often the kinds of problems you face, an RL system is not going to find that (sorry!). It’s still perfectly reasonable to have parameters in your system that you tune live (in fact this is a very good idea that many firms use), but this is a simple gradient descent problem, not much more. Same with the forecast and spread part, you can optimize this using traditional methods and get much further than some fancy setup. You also will run into lots of issues with accurately simulating the market which make it a complex system to set up for RL approaches (not to mention the compute cost of the NNs) which all add up and cause it to be a nerdhole that isn’t worth it for the vast majority of trading firms.

Anyways, this is my rant as to why I think most of the market making literature is bs.

### Machine Learning

---

The core issue here comes again from a misunderstanding of where alpha actually comes from. In most cases you will have a pipeline that looks something like this:

[![](images/e2e1e95158b4.png)](images/29221bccf390.png)

Some people (not very profitable people) skip the feature step and leave that all up to the model. In other fields, this works, but not in quant (the NN finds a pretty terrible set of features)!

You are effectively applying transforms. Starting with data, then into a feature (which describes some pattern in the data), to a forecast of future price (through your ML model), to an optimal portfolio (via portfolio optimization), and then finally into a set of trades (through your execution system). Most of the “edge” is stored in the feature part of things. Data is simply something to not mess up, and the rest of them are multipliers.

Say you have a great ML system v.s. a basic one, perhaps that is a 1.2x multiplier instead of a 1.1x multiplier; it’s an extra 10%. This is HUGE if you work at Citadel and have hundreds of alphas which make hundreds of millions. Same if you manage to improve your portfolio optimizer or execution such that your PnL increases 10%, and this is why they have huge teams which are dedicated to this and spend lots of time on nothing but these activities.

However, if you are a small shop then you have a choice. Either increase the performance of all your alphas by 10% or find another alpha. If you have 2 alphas you get a 50% gain in your edge by finding another alpha. Until you reach the scale of large shops, it really doesn’t make sense to spend lots of time optimizing your machine learning models.

Regardless of this, lots of beginners seem to believe that applying neural networks and complex forecast models to things is where the alpha is at. Aside from the fact that these usually just end up overfitting and rarely beat XGBoost out of sample (or even just a ridge regression), they still don’t make sense to develop even if many of these did work. On that note, it takes a lot of time to learn how to build really successful advanced forecasting models and it’s a niche within this field that has very few jobs available. There’s a lot more jobs making the features than there are optimizing those features, and it’s often guys from AI backgrounds that get invited to do those roles.

### Models vs. Features

---

The diagram from the prior section is not what the pipeline always looks like, and sometimes it is entirely informed by a model itself.

An example of this is pairs trading based approaches which are almost always model informed (unless you try to make some sort of average pairs distance as a cross sectional feature). For me this was fairly successful when it did work but it also took ages to test each idea owing to the complexity of implementation for each.

These sorts of model informed strategies, whilst they can still be successful, take a lot longer to produce and generally offer similar rewards as defining simple IF logic or features which represent your ideas. So when you take into account the huge time cost which is often 10x+ that of simpler ideas then you start to see why it is so rare that they make sense to prioritize.

Work on ideas that can be iterated over very easily for maximum alpha output, no single idea is worth fixating on so much that it costs you a month of your time. HFT models can be somewhat of an exception but this is usually because it isn’t one single model — it’s a collection of many advances which all start with a simple test in prod showing there’s money in the trade. When it’s theoretical and located in the world of backtests then we are playing a different game.

### Time Optimization

---

I think this is one that is extremely important when it comes to doing effective work and it’s one of the things you learn only as time progresses. Time optimisation is key, and I’m not talking about avoiding distractions and maximising the time at the desk where you are focused - that actually doesn’t have a lot to do with what I’ll be discussing (although I’m sure it does help and I’ve certainly improved at this with age).

Instead, you should optimise the work you are doing and find ways to make it occur faster. You often don’t need model everything. Often there are much simpler ways to get the answer you need without whipping out a Monte Carlo + Empirical Distribution with all sorts of complex modelling when you can do a dirty test of the problem using just 1h trade bars.

There’s often a much simpler way to get roughly the right answer. Think about whether it needs to be precise? It usually doesn’t need to be precise and even a rough fit of the parameters for a model will yield results if your ideas make sense at a fundamental level / have alpha. Prioritise getting results over exact solutions that take much longer than the dirty way.

### Dashboards

---

This is something that I’ve seen with both beginner quants who have no idea what they are doing and quants who have full on jobs at top tier funds and are meant to know what they’re doing… they love to make dashboards.

I’ll start with beginners. It’s often a beginner project I see where they make some sort of financial analysis dashboard. This basically teaches you nothing about quant. You’ve only managed to take some price data, and plot it in a nice UI. The skills you have learnt include:

- Plotting
- GUIs
- Perhaps some basic data streaming.

These are basic Python skills that are not quant specific. If you aren’t very good at Python then sure, go ahead and learn it, but don’t go ahead with a project like this thinking it somehow teaches you a quant specific skill. Making “quant dashboards” is not a good project unless you are trying to get better at the language itself.

As for quants that know what they’re doing in their chosen language and are often employed in junior researcher roles, this is just another kind of nerd hole. Your firm doesn’t need to see your work presented in an aesthetically enjoyable and feature rich trading dashboard - they just want to see if it makes money and the results of the analysis. You should still make your graphs legible and label your axes (date ranges instead of index ranges on X-axis is a nice presentability tip), but making things pretty doesn’t make money and in my view this is another type of nerd hole that people eventually learn is not a wise way to spend their time.

Anyways, that’s all for this point, but just remember - making things pretty and graphically enjoyable doesn’t make money — especially when it’s only ever going to be used by a handful of people!!!

### Cynicism

---

You will learn not to trust any positive result in your backtests one way or another, but at the start this reflex that most experienced quants have is almost always missing. It is all too common that beginners will find incredibly smooth PnL curves and think that they’re going to be incredibly rich (this is one I relate to at the start of my own career), but your first reaction when there is a result that looks too good to be true is to expect that there is some sort of lookahead or otherwise a research error. I have already written an article on the many ways you can create errors in your research process for those curious:

[![Why is my backtest wrong?!](images/65f4e0492a4c.gif)Why is my backtest wrong?![Quant Arb](https://substack.com/profile/101799233-quant-arb)·March 22, 2025[Read full story](<https://www.algos.org/p/why-is-my-backtest-wrong>)](https://www.algos.org/p/why-is-my-backtest-wrong)

Being extremely cynical about your own results and figuring out as hard as you can what got messed up (there’s usually an issue when the result is super good) is one of the skills that is necessary to produce great research. I can only tell you to be cynical so much though, and I do think that the instinct for when something looks off or generally too good to be true does come with time for at least some significant part of the reflex.

### Nothing Beats Doing It

---

This is more for those who are learning and I think it applies broadly across industries. There is nothing like actually going out and doing the work. Not only is it hard to verify who knows what they’re talking about at the start (an issue that is especially relevant in trading / quant when there are tons of grifters + academics who have never made money trading), but more than that, even if you do find lots of useful information, you will still forget most of it unless you use that information.

Using that information does not explicitly include doing research since you can also talk about the findings with others and this is great for learning / critically thinking / connecting dots, but there also comes a time when you need to test this knowledge. Taking us back to what I discussed earlier with neural networks - they’re extremely popular in the quant discourse but they really don’t make much money unless you are at a top firm and have the time to specialize in just that + the scale where that small an improvement makes a real difference in USD terms (enough to pay your salary). Simply talking about it will probably eventually lead you to the right answer, but if most of those who you talk about it with are similarly experienced (which will often be the case as a beginner) then there’s always the possibility of your own naïvetés between yourselves feeding into each other and pushing you further down nerdholes. This was something I can say I experienced towards the start of my career at least. There was a lot of mention of methods that didn’t make sense but because others I knew talked about them (who frankly knew no more than I did) it became normalized. It was only when I actually tried to make money with them that I realized they were not so useful.

So to sum this up, put the textbook down, open up a Jupyter notebook and implement something. There’s plenty of further work and examples I have given in my small trader alpha series so take a crack at one of them, they’re great starting points.
