# Finding Arbitrage Opportunities

Source HTML: [`html/2025-09-22-finding-arbitrage-opportunities.html`](../html/2025-09-22-finding-arbitrage-opportunities.html)

# Finding Arbitrage Opportunities

| 항목 | 값 |
| --- | --- |
| 날짜 | 2025-09-22 |
| 접근 | 유료 |
| URL | https://www.algos.org/p/finding-arbitrage-opportunities |
| 부제 | Where the best arbitrage opportunities are |

---

[![Super Dump Of Vintage/Retro Science Fiction Art - Imgur](images/b073b7a40714.jpeg)](images/792c3bd56690.jpeg)

### Introduction

---

Half the work with arbitrage trading is figuring out where the opportunities are. You can make your system as advanced as you want, but if you can’t find the exchanges and products where all the returns are concentrated then you won’t be very profitable.

In this article, I explain what to look for when deciding on whether to add an exchange and even when you do integrate an exchange the reasons why the initial research showing arbs could be unrealistic (wash flow, internalized flow, etc).

Then, finally — the part most of you are deeply interested in, I explicitly share where the best arbitrage opportunities currently are and can actually be captured based on my own private research.

### Index

---

1. Basic Integrations
2. Wash Flow
3. Internalized Flow
4. Geographic Constraints
5. Exclusive Deals & The VC Edge
6. New Exchanges
7. Reward Farming
8. New Products
9. Where I think the best opportunities are

### Basic Integrations

---

Before we go about integrating exchanges where we think there will be an inefficient market we actually have to start with markets where it is efficient. If we are hedging, we need ultra liquid / competitive exchanges where we can hedge our positions in a cost-effective manner. Even if you don’t plan on hedging you will be deriving your fair value from top name exchanges and hence will need to have feeds for these exchanges to get their live prices.

We can call these the basic integrations. Even if you don’t actually plan on finding alpha there you still need to add at least Binance and perhaps even others for more advanced approaches in order to get the best possible fair value.

With that in mind, I will now start with what are two common pitfalls when searching for exchanges that will waste all of your time.

### Wash Flow

---

One of the first and most important lessons you will discover is that just because an exchange says it has lots of volume doesn’t mean it actually does.

Many of the exchanges have lots of wash flow present which makes any arbs you find on them impossible to actually trade because the quotes aren’t real (or if you are trying to get a maker fill, the trades are not real so there is no flow to fill you against - it will just go to mysterious and non-existent levels, breezing right past your quotes).

One easy inspection for wash flow is checking that open interest and volume match. Often they only bother to spoof volume and not open interest (which is far harder to spoof). That said, these exchanges are brazen and will just put out fake volume and open interest numbers, they aren’t even trading against themselves and ensuring their own fills - the trades simply never happened.

For the exchanges that aren’t committing outright fraud, you can often measure the % of trades that fill inside of the spread. Trades that fill inside of the spread are extremely rare and usually occur because 2 people sent opposing orders at the exact same time. On exchanges with wash flow you’ll see the % of orders filling inside the spread above 10-20%, sometimes even at 90%+. It’s usually less than a single % on reputable exchanges.

Loris has a few good tweets on using DNS to detect when an exchange is faking it as well:

<https://x.com/0xLoris/status/1969197574570549697>

<https://x.com/0xLoris/status/1965436474691625196>

### Internalized Flow

---

You often will find exchanges where they appear to be great places to trade (and often are near the top of exchange rankings) but they’re basically exclusively for retail. That’s because the market maker on their exchange is the exchange themselves.

When all the flow is super uninformed it’s very profitable to simply take the flow on yourself instead of letting others MM it, and hence some exchanges will do this. MEXC for example simply shut off their futures API for 9 months because they didn’t like the flow.

It depends how reputable they are, some will just not give you api access, and no volume discounts but others will simply refuse to give you your profits on the grounds of manipulative trading, and even if you do get it back they will ban you afterwards.

### Geographic Constraints

---

Sometimes there is an edge in being a local in a country where the exchanges aren’t very accessible. An example of this is Korean exchanges. These tend to have really good opportunities and the top names with all the opportunities pretty much never approve institutional accounts (mostly because they’re the market maker on the exchange) so if you have a Korean passport and can create an account you can have access to this exchange and pick them off.

Despite my earlier comments regarding internalized flow, there are still often cases where even if you are trading against their internal desk you can get away with it and as long as you don’t admit to arbing them then they’ll let you get away with it. There was one exchange I used to trade on where we were >80% of the taker volume for a period and were just aggressively picking off their internal MM. They got a little wiser but we still continued to print there… so there’s definitely cases where despite trading against the internal desk it can be very profitable.

### Exclusive Deals & The VC Edge

---

This is incredibly profitable, and I have experience with this first hand. Back at a previous firm of mine we were raking in 6 figures every single month (in addition to getting millions of $$$s of equity options in the exchanges shares) on an extremely high Sharpe simply from marking up a leading exchange by a fixed spread and quoting it. We were 90%+ of the volume and had been there largely because we started quoting when the exchange was in its infancy due to our VC investment in the exchange. I would bet that the firm is still making this kind of money to this day doing just this…

There are many firms that do this, but one of them is Jump crypto. There’s a reason they’re so eager to invest in crypto exchanges and often lead rounds. It’s because it aligns with their trading business and let’s them get into preferential positions.

If you aren’t extremely well capitalized and don’t have a VC arm then there are still ways to do this, but you need to network with the exchange founders to become the designated market maker from the start. I’ve known people in the space who traded books based around this general idea. One of them just had a single exchange that they market made for and that was it - they weren’t making stupid money but it was clearly enough for them to be very happy (they hired quite a few new people off the back of that PnL and eventually expanded away from it).

### New Exchanges

---

It’s usually on newer exchanges where you find the best arbitrage opportunities. Old exchanges tend to have been explored and there’s plenty of time for fellow arbitrageurs to find out about whatever opportunities exist there. So it is typically a requirement to find good arbs that the exchange be at least fairly new.

Whilst there are many tales of people doing well on Hyperliquid etc, I don’t think this is where the money is, and it hasn’t been there for a while. It’s now a main name exchange and there aren’t going to be any easy arbs. The money on well integrated names like Lighter xyz and Hyperliquid requires very advanced trading systems and is probably more on the maker side than the pure arbitrage side.

### Reward Farming

---

This is an extension to arbitrage because it usually is accompanied by arbitrage trading or market making around Binance price. Sometimes an exchange will have a stupidly good reward program where they are basically handing over VC (or DAO, or their own token) money. An example of this (which I tweeted about multiple times at the time of it !!!) was Lyra — the taker fees were 1/5th of the reward on taker volume. It isn’t very hard to figure out what the answer is here. Just churn volume. The maker rewards were also just as juicy so if you traded back and forth (whilst trading properly because they did revoke rewards for a couple wallets for wash flow churning - although these guys were super blatant) then you could do extremely well.

Staying on the lookout for these sorts of opportunities are wise because they are the most profitable by a mile. In fact, sometimes exchanges will offer you into the 10s of thousands per month to integrate as a market maker, although being one of the people who gets those offers requires you to be a bit more than a solo quant. I’ve been offered ones between $45,000-75,000 before so very juicy if they consider you professional enough to let you in on the programme.

### New Products

---

Sometimes an exchange will roll out a new product and it will create lots of opportunities like when Bybit rolled out spot trading. When it launched it was fairly illiquid but the flow was really good since Bybit was a HUGE futures exchange and hence had tons of retail flow to send towards the spot markets. Until it became well integrated (and now is one of the biggest spot markets of course) it was a great place to trade.

### Where I think the best opportunities are…

---

I’m sure this is the most interesting part for everyone, and is what most readers will skip to. Integrate lots of DeFi exchanges, even ones with AMMs - the shittier the better, and you’ll find tons of arbs. Especially, on the Solana exchanges. Anything where the API barely works is often loaded with arbs when it comes to on-chain.

That’s it - thank me later :)
