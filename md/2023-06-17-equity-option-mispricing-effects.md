# Equity Option Mispricing Effects

Source HTML: [`html/2023-06-17-equity-option-mispricing-effects.html`](../html/2023-06-17-equity-option-mispricing-effects.html)

# Equity Option Mispricing Effects

| 항목 | 값 |
| --- | --- |
| 날짜 | 2023-06-17 |
| 접근 | 유료 |
| URL | https://www.algos.org/p/equity-option-mispricing-effects |
| 부제 | Interesting opportunities for profit in equities options |

---

[![Derivatives Images – Browse 22,103 Stock Photos, Vectors, and Video | Adobe  Stock](images/f1eafb4eda23.jpeg)](images/beabf06c67cd.jpeg)

#### Introduction

---

This short article will explore a few mispricing effects that have been documented for equities options markets. We will cover some effects that are probably already familiar to some readers and also a fair few that are lesser known. Hopefully, this points you towards some good ideas for developing strategies in this area, and I’ll try to add a bit of insight along the way.

#### Index

---

1. Introduction
2. Index
3. PEAIV
4. Growth Premium
5. IV vs. RV Effect
6. DHOR vs. Idiosyncratic Volatility
7. Distress Premium
8. Option Momentum
9. Risk-Adjusted Option Momentum
10. Delta Hedging

#### PEAIV

---

This is probably one of the best-known mispricing effects, and as a result, it is very hard to beat fees with nowadays.

Simply put, IV increases pre-earnings announcement and decreases afterward.

We can further add in some alpha by trading the PEAD (Post Earnings Announcement Drift) with our options position. This is only for when we are trading the short volatility side of things.

Selling naked options is generally a bad idea, so whilst we can use straddles as a cheap way to be long volatility, we will probably want to use butterflies to get short volatility. Thus, if we have a large surprise on the earnings (cause for PEAD), we can capture the volatility crush post-earnings whilst also playing the momentum via an asymmetric butterfly where we hold less of or no options on the call / put side to get directional exposure.

#### Growth Premium

---

There is a strong premium on call options for growth stocks. This can potentially be viewed as a premium on skewness, but my view is that retail buying these up is likely the biggest cause. Hence, we see this premium become greatest when looking at 0DTE options, which are a fan favorite for retail traders (even after adjusting for the higher premium on short-dated options).

#### IV vs. RV Effect

---

When we compare the historical volatility vs. implied volatility, we find that the relationship between the two is a strong predictor of DHOR (Delta Hedged Option Returns). We use the delta-hedged returns to remove as many dependencies on the underlying as possible, and this gives us a much more well-behaved distribution once we’ve done this.

#### DHOR vs. Idiosyncratic Volatility

---

Another interesting predictor of delta-hedged option returns is the idiosyncratic volatility of the underlying instrument for the option.

Much like the prior effect, we can frame it as a risk premium, but a much better way to approach it is by asking, “When is this premium greatest?”. Capturing risk premiums in a diversified manner and doing so when they are the greatest is a robust way to generate excess returns across all markets.

#### Distress Premium

---

This, much like the growth premium, can be viewed as a lottery effect where people are willing to overpay for options that behave more like lottery tickets than anything else. I’m sure there are plenty of psychological explanations behind this one along the lines of people overestimating the likelihood of very unlikely events, but generally, there is a premium on call options for distressed firms.

The “lotteryness” effect is usually regarded as its own premium, both for equities and options, but it overlaps heavily with the growth and distress premium. The idea behind them is generally consistent across these.

#### Option Momentum

---

Momentum in option returns is far more statistically significant than it is for equities and comes without the momentum crashes or long-term reversal effects. This is an effect that generates 3+ Sharpe pre-transaction costs, but as we see in the next section, it is entirely subsumed by risk-adjusted option momentum.

As with the previous effects I’ve listed, we use delta-hedged option returns for this momentum.

#### Risk-Adjusted Option Momentum

---

This is probably the most profitable effect out of them all. It generates a high Sharpe ratio post-transaction costs (3+) and has a 5+ Sharpe ratio pre-transaction costs.

Using the risk factors for options identified in Bali, Beckmeyer, Moerke, and Weigert (2023) and the risk factors for equities identified in Jensen, Kelly, and Pedersen (2021).

We apply Instrumental Principal Component Analysis to deal with the fact that option characteristics change with the underlying as their moneyness evolves.

A simple momentum signal is adjusted by the risk exposures to the largest factors, and this is then traded to produce a highly profitable options momentum strategy.

The paper “A New Option Momentum: Compensation for Risk” gives more details on the implementation for those looking to implement this.

#### Delta Hedging

---

Most of the papers so far have used daily delta-hedged option returns, but this isn’t exactly the most optimal way to do things. At the end of the day, delta hedging is a trade-off between risk and transaction costs. Time doesn’t have much of a place here, so an optimal delta hedging solution shouldn’t include it.

Minimum Variance Delta (MVD) hedging is explored in the below paper:

<http://www-2.rotman.utoronto.ca/~hull/downloadablepublications/Optimal%20Delta%20Hedging.pdf>

And the trade-off between risk and transaction costs is solved in this paper:

<https://efmaefm.org/0efmameetings/efma%20annual%20meetings/2005-Milan/papers/284-zakamouline_paper.pdf>

Often it is easiest to use a simple method, and Zakamouline et al. is going to be a lot less work if you aren’t planning on making delta hedging the focus of the research (as opposed to a necessary step in researching option mispricing effects).
