# How to level up your arb game

Source HTML: [`html/2024-04-07-how-to-level-up-your-arb-game.html`](../html/2024-04-07-how-to-level-up-your-arb-game.html)

# How to level up your arb game

| 항목 | 값 |
| --- | --- |
| 날짜 | 2024-04-07 |
| 접근 | 유료 |
| URL | https://www.algos.org/p/how-to-level-up-your-arb-game |
| 부제 | The paths to improving an existing arbitrage trade |

---

[![Visualize a trading algorithm represented as a futuristic robot, surrounded by dynamic digital effects and icons symbolizing stock market elements like graphs and currencies. The scene is set in a virtual space, resembling a video game, where the robot stands on a podium that glows with neon lights. Above the robot, a large, glowing 'Level Up!' sign hovers, emphasizing the video game-like progression. The background is filled with abstract digital patterns and binary code streams, highlighting the high-tech environment. This image captures the moment of the algorithm advancing to a new level of capability and performance.](images/5637a4d4c433.webp)](images/9cc31af2ca2c.webp)

#### Introduction

---

Especially when it comes to the high-frequency world, trades quickly become crowded and die out. I’ll list out some arbitrages that used to work in their basic form and then died off:

1. Triangular arbitrage
2. Perpetual basis arbitrage
3. BTC lead-lag
4. Spot / spot arbitrage

This extends past pure arbitrages to more statistical trades or event-driven ones.

Of course, how you optimize a trade varies based on its attributes. A longer-lasting statistical alpha gives more room for execution enhancements than an event-driven alpha like Elon tweeting about DOGE, where the alpha realizes over a very discrete interval.

In this article, we dive deep into how to level up your alphas at very broad level. For specific examples where I’ve walked through this for individual trades, my small trader alpha series gives clear examples of many of these trades (Funding arbitrage, Spot/Spot, Triangular) which with the exception of funding arbitrage is dead in the traditional form, but as we show is still very profitable in modern day by levelling up the trade.

#### Index

---

1. Introduction
2. Index
3. Execution
4. Statistical
5. Exposures
6. Universe Expansion
7. Scope Expansion
8. Latency
9. Trading Costs
10. Leverage
11. Walkthroughs

    1. Funding
    2. Triangular
    3. Spot
12. Final Comments

#### Execution

---

It’s often the case that we see teams evolve their HFT strategies over a common path, in these stages:

1. Taker / Taker
2. Maker / Taker
3. Maker / Maker

This is how arbitrageurs become market makers so frequently because they start off with a great trade that works fine. Then, as they expand it, there begins a transition to more making in the execution, until eventually the alpha dies and they are left with a full maker system that can no longer be called an arbitrage strategy and is now a full-fledged market-making strategy.

If you have an arbitrage that involves taking, you’ll start by making the first trade using a limit order and entering the maker/taker world. This is easier than all maker because you still ensure the trade remains fully hedged throughout.

With taker/taker, you control both legs, so you can ensure that you hedge after the first leg completes, and with maker/taker, whilst you don’t control the first leg, you can still make sure that the remaining legs will complete.

Maker/maker becomes a little bit more complicated, but by the time you transition to it, the general concept should be quite easy because, hopefully, you’ve solved the core optimization problems related to maker/taker already.

The main thing to consider is the signal decay. If your signal lasts a very long time, then you can afford to make into it. These are arbitrages that are uncompetitive since the exit liquidity on the exit legs will not be taken rapidly.

There is then also the question of what causes the discrepancy. If you get filled on a limit order that is 5% wide and then cross-exchange exit that trade, then it was probably the insanely wide limit order that created the profitability.

However, if you earn 10 bps on an arbitrage where the difference in midprices cross-exchange is 5%, then you’re still earning all the profits from the difference in midprices.

Thus, it’s important to establish how much of the actual edge you are gaining from this maker model. This isn’t just about the decision of how valuable making is, but more about alpha decay. Discrepancies in price tend to resolve a lot faster, but of course, if you are getting filled to make the edge, then it’s simply an analysis of your fill times.

Here’s what you’ll need to solve:

1. Optimal level of spread (trading off how much you get filled vs.. spread earnt)
2. How aggressive should I be based on signal decay (the balance between the signal going away over time and you getting better fills if you wait longer / wider)
3. Quoting constraints (spot requires inventory to quote, and margin considerations will all play a role)

#### Statistical

---

There are many ways to improve a trade on the statistical side. Here are some examples of how we could do this:

1. Persistence modelling (how long will this funding arb exist or how long will this triangular arbitrage exist - this will play into execution as well and how you set your limit orders)
2. Removing unnecessary legs (Can we only take one side of this triangular arbitrage or cross-exchange trade and make it a lead-lag trade)
3. Pairs trading the spread (Instead of cross-exchange settling, we can just pairs trade the spread and exit on both exchanges)
4. Analysis of opportunities (Do we get picked off more on large caps than small caps? Can we analyze our trades so far and find common themes between the most profitable ones. Not all arbs are risk free, your orders can be picked off or you can be too slow, or it can take ages to converge, what predicts all of this?)

This is a non-exhaustive list but it provides a start on how we can improve trades. All 4 of these points apply to almost every trade I’ve explored. Post-trade analysis is key and you’ll often find that certain assets or exchanges for some reason have no competition and you have great persistence etc.

#### Exposures

---

People often try to be delta-neutral when they make arbitrage trades, but holding inventory and running deltas is a great way to improve.

For triangular arbitrage, if I want to quote limit orders on the sell-side of a spot asset, I need to hold inventory of it first.

If I am doing spot arbitrages between exchanges, it may make sense to hold more spot inventory before transferring, as transferring has a fixed cost.

Funding arbitrage sometimes makes sense to hedge in a correlated asset when it’s too expensive to get the actual underlying.

All of these come with risks that can go massively against you so it’s important to understand the risks and whether they come with a premium - some don’t.

The cases where there is no premium on the risk, you may want to find alternative ways to take off risk. Do you find that an arbitrage flips both ways? Perhaps hedge in the future because it is cheaper and eventually this should all balance out. Hedging in futures for spot cross-exchange arbitrages can be a great way to protect yourself against scenarios like LUNA, where the arbitrage exists because no one wants to hold it due to it crashing. Identifying scenarios like this again comes back to the statistical side of things.

#### Universe Expansion

---

Expanding the universe of the trade is an easy way to make more money. You should make sure to trade every asset on the exchange.

This sounds quite trivial, but often people only manually select assets to trade, and do not have the automation necessary to truly trade everything.

Oftentimes, the systems cannot handle the data input required to trade large universes as well, but all of this should be planned ahead.

Build a quick demo to test the arbitrage ASAP (MVP), and then from there, think about scale.

#### Scope Expansion

---

Scope expansion really just means adding new assets to the universe of what we are trading that change the scope of the arbitrage. This is because the assets are not the same as the existing assets in how they are dealt with practically.

Here are some examples:

1. Adding coin-margined futures to a perpetual basis trade against USDT-margined futures.
2. Adding assets from another exchange
3. Adding dated futures with an expiry to the basis trade with a model for normalizing them with confidence included
4. Adding spot assets to a futures-futures funding trade to do both spot vs. futures and futures vs. futures in one place.

I want to highlight #3 as an example of a situation where statistical intervention is actually required. In fact, they are so different that our normalization process becomes guesswork.

Scope expansion almost always will involve some statistical elements because of this. It is rare to get a direct translation between assets with no caveats. Even for spot vs. spot cross-exchange, we should think about the probability that holding inventory is a premium simply because the asset is plummeting.

#### Latency

---

We can always look at improving our latency as a way to stay competitive as well.

[![Low Latency Dup. Data Aggregation](images/5d7908bb5349.png)Low Latency Dup. Data Aggregation[Quant Arb](<https://substack.com/profile/101799233-quant-arb>)·October 8, 2023[Read full story](<https://www.algos.org/p/low-latency-dup-data-aggregation>)](https://www.algos.org/p/low-latency-dup-data-aggregation)

In the above article, we give a comprehensive guide to latency reduction, but latency goes beyond the actual system latency and into identifying when latency will spike.

This is on the statistical side again, but we can often find proxies for things like:

“if the deviation is smaller than X, we are unlikely to be fast enough, but if it’s larger than X, then we will be fast enough”

This could also be volume-based. These are real ways that I have improved trades in the past.

#### Trading Costs

---

This can be done in 4 ways:

- Prime Brokers
- Negotiation
- Churning
- Crossing

The first, second, and third way to reduce trading costs come from improving your fees on exchange.

The last is by crossing your flow against other flows.

To start, you can talk to a prime broker who will give you a subaccount to trade out of and offer you much better fees. You won’t have the best fees possible since they’ll take a little for themselves, but they always offer to improve your fees unless you have the best or near the best already. They can also be negotiated with. The thread I wrote below is a bit old, but worth looking at if you want to understand the space:

[Prime Brokers Thread](https://x.com/quant_arb/status/1612265350506086400)

To give an updated recommendation, I would say LTP is probably the best to go with currently, although with the recent regulatory issues it very much depends on your situation. They won’t touch anything US related, but here are your options generally:

1. LTP (Liquidity Tech)
2. FalconX
3. Matrixport
4. HRP (Hidden Road Partners)
5. Copper

For the negotiation front, this comes down to how well you can convince an exchange you are a major player in the space and worth cutting a deal with OR how much you can trade on something they want to grow. Often, they’ll have products like combo books that they’ll give you special incentives to trade on, and you can work those into better fees.

Churning volume intentionally to grind fee tiers is another common way to go about it, and you can include the value of volume itself in your trade logic to get there.

Finally, if we have multiple other strategies in operation, and they decide to trade at the same time, we can match their orders internally before sending them out. This reduces our trading significantly, and is a technique I’ve talked about extensively in previous articles.

#### Leverage

---

There are 4 main ways to acquire leverage:

1. Built-in exchange leverage
2. Custom exchange leverage
3. Prime broker leverage
4. External leverage providers

For most exchanges, you can borrow spot or for the perpetuals you can get incredibly levered. This is enough for most people and represents the most basic form of leverage that everyone has access to. You will also get better spot borrow rates based on fee tiers for many exchanges.

Many exchanges will also offer zero-interest loans (which do need to be collateralized and come with LTV requirements) if you generate over a certain volume. These can again be negotiated once you start moving size.

Prime brokers will offer leverage as one of their services, and you can discuss with them how this can be organized. With all the regulatory scrutiny nowadays, it tends to be a bit more case-specific.

Finally, firms like Tesseract or Cicada can take a look at a trading firm’s balance sheet and write loans based on this and the perceived risk they gather from that balance sheet, which is much more flexible in terms of how they can be deployed. These firms are usually able to move quite fast, but they still have a fair bit of operational work in terms of contracts and agreements.

#### Walkthroughs

---

In this section, we will walk through each of the sections we defined for different trades (funding, triangular, spot-spot), and outline the specifics of how it applies.

You can also check out previous articles in the small trader alpha series (with more to come in the near future) for a much more in-depth dive, with relevant statistics, charts, and analyses included.

**Funding Arbitrage:**

- **Execution** : For execution, we have 2 legs and thus can start with making into the first leg, and making out when we decide to exit. This is very standard other than the fact that whether to enter / exit is often swayed heavily by cost (i.e., break-even time for collecting funding is 14.5 hours so we can’t turnover too much, but if we quote super wide and get a great fill then maybe that drops and we’re willing to exit now because the spread has converged a bit and better opportunities exist)
- **Statistical** : Persistence prediction is a core part of this, as is modeling risks around the spread blowing up cross-exchange or even statistical alphas related to what happens when we hit funding rate caps, etc., and whether this can become a trade.
- **Exposures** : Holding basis risk, holding the risk of margin calls cross-exchange, especially if we start mixing spot vs. futures on risky shit, hedging in correlated assets, hedging overall margin betas cross-exchange (i.e. if I am long 10 ETH betas (betas as in worth of ETH exposure, this could be WIF etc) on Binance and short 10 ETH betas on Okx, I put on a 10x10 ETH spread since if ETH moves a shit ton I don’t want to get margin called from everything lead-lagging and ETH is cheap to trade compared to shitcoins).
- **Universe Expansion:** Trade all tickers
- **Scope Expansion:** Trade coin-margined, trade quanto, trade cross-exchange, trade spot & futures, trade expiring futures, trade USDT or USDC or USD margined. Use DeFi borrow. Many options.
- **Latency** : Maybe you might have latency in terms of fighting to get the borrow or for moving your quotes when making but otherwise this is unlikely to be an issue.
- **Trading Fees** : Nothing specific, see the section on this for a recap.
- **Leverage:** Keep in mind APRs and perpetuals give you better leverage, but be smart when it comes to your exposures. Model your risk with scenario analysis. Lost too many homies to stuff like this :^(

**Triangular Arbitrage:**

- **Execution** : This is probably the fastest resolution in terms of how discrete the alpha is, so there is a lot of debate on whether making it is always the best route. You definitely need a model that decides whether to make or take; don’t just assume one. There are 3 legs; it probably isn’t that worthwhile to make on the last leg since it’ll be something like BTC/USDT or ETH/USDT, but shitcoin to USDT and shitcoin to BTC (or ETH) are probably worth making into.

Giving away some alpha: I found it was worth quoting a multiple of the taker liquidity in the book, then, if filled, taking most of the liquidity in the book, but then making the rest of it that remained. It was more of a booster of tight liquidity available than a cost improvement.

- **Statistical:** Does this go both ways? Should I hold inventory and cancel it out when it goes back? Can I take one leg of this as a bet? How does this affect futures?
- **Exposures:** Need to hold inventory to quote the sell side of spot assets because borrow is too expensive. Exposures when transferring cross-exchange. Exposure to the arbitrage closing when doing cross-exchange if one leg requires inventory that is transferring.
- **Universe Expansion** : More tickers
- **Scope Expansion** : More exchanges
- **Latency** : Spamming limit IOC's at levels that would create an arbitrage in case the price moves there you get hit first. Many ways to optimize outlined in my latency optimization article.
- **Trading Fees** : Nothing specific, see the section on this for a recap.
- **Leverage** : Again, nothing too specific. We may want to use external lenders because Spot is not very capital-efficient, whereas for funding, arbitrage futures are very capital-efficient. You can use margin trading mode and borrow USDT, but this is really expensive.

**Spot-Spot Arbitrage:**

- **Execution** : This is an in-between of funding arbitrage and triangular arbitrage. You don’t have the same amount of timing flexibility to get into the trade as with a funding arbitrage, but you are still not at the level of pressure a triangular arbitrage has.
- **Statistical:** Does this go both ways? Should I hold inventory and cancel it out when it goes back? Can I take one leg of this as a bet? Can I hedge with futures? Is this an arb or a premium because it is plummeting? Optimal transfer point.
- **Exposures:** Need to hold inventory to quote the sell side of spot assets because borrow is too expensive. Exposures when transferring cross-exchange. Exposure to the arbitrage closing when doing cross-exchange if one leg requires inventory that is transferring.
- **Universe Expansion** : More tickers
- **Scope Expansion** : More exchanges
- **Latency** : Many ways to optimize outlined in my latency optimization article, but apart from moving limit orders or any immediately taking on exchange parts, there will be a multiple minute delay during the transfer that can’t be resolved without borrow access (which is expensive).
- **Trading Fees** : Nothing specific, see the section on this for a recap.
- **Leverage** : Again, nothing too specific. We may want to use external lenders because Spot is not very capital-efficient, whereas for funding, arbitrage futures are very capital-efficient. You can use margin trading mode and borrow USDT, but this is really expensive.

#### Final Remarks

---

There are many ways to improve a strategy, and this should serve as a basic overview that tries to generalize these ways. This is by no means exhaustive and there are often very specific upgrades to each strategy that reflect the unique nuances of that strategy.

I hope this was a good read; enjoy the day!
