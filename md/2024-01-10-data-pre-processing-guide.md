# Data Pre-Processing

Source HTML: [`html/2024-01-10-data-pre-processing-guide.html`](../html/2024-01-10-data-pre-processing-guide.html)

# Data Pre-Processing

| 항목 | 값 |
| --- | --- |
| 날짜 | 2024-01-10 |
| 접근 | 유료 |
| URL | https://www.algos.org/p/data-pre-processing-guide |
| 부제 | Avoiding key mistakes and getting the most out of your data |

---

[![cover photo](images/2169cb3edae2.webp)](images/cacca0081811.webp)

#### Introduction

---

Before any research can be conducted, we must prepare our data and, in doing so, transform it into the most useful version of itself. There are two reasons here as to why we need to pre-process our data:

1. Removal of errors (accuracy)
2. Structuring, reduction, & transformation of data (makes analysis efficient)

In this article, we walk through best practices for each of these areas/reasons behind data pre-processing with the aim of improving readers' research abilities and hopefully avoiding common mistakes that beginners and even more experienced practitioners often make.

#### Index

---

1. Introduction
2. Index
3. Ensuring Accuracy
4. Transformation and Reduction of Data
5. Final Remarks

#### Ensuring Accuracy

---

Accuracy can take many forms beyond ensuring the data’s values are correct. We need to ensure that our data is not misleading us subtly. This section goes beyond the data being wrong to common issues faced because of misuse or misdirection in/of the data as well.

***Bid/Ask Bounce:***

This phenomenon occurs when the price bounces between the bid and ask prices. Most providers will form their candlesticks from trade prices and not from mid-price. This leads to issues where we observe the close price oscillating up and down on illiquid assets by large amounts and perceive this as mean-reversion. Thus, making all candlesticks out of mid-price data is generally considered wise, which typically has to be done manually as vendors don’t provide this.

***Aggregated Multi-Venue Prices:***

Especially in markets with a high degree of fractured liquidity or differences in trading costs (for various reasons), we may see vastly different prices when comparing venues. This may not necessarily be arbitrage-able since these prices typically are not inclusive of fees, but when prices are formed from the aggregation of multiple venues, you can find:

1. Massively inaccurate high - low price differences.
2. False mean-reversion as the price oscillates between the price on two different venues.

***OTC Price Inclusion:***

Some exchanges, especially for digital assets, will include OTC trades in the main trade feed, so errors can occur when these trade prices do not match the calculated best bid/ask. This usually creates issues related to assuming the current state of the orderbook. Trades are typically sent out before quotes are, so when trades occur, people will also use this information to update their orderbook. This creates problems when the trades don’t actually impact the orderbook due to their OTC nature - creating a momentary false perception of the orderbook.

***Trade Aggregation:***

Whilst this does not necessarily create issues when used properly, there are mistakes that can easily be made when researchers do not differentiate between individual trades and aggregate trades. If I send an order for 1000 units and 100 matches at the first level, I will create one trade for 100 at the first level of the book, and so on, for every price my order impacts through. Some exchanges / vendors will show each trade, whereas others will simply show the average trade price and aggregate them all as one.

It is important to know the relevant kind of trades to use. If we are detecting TWAP algorithms, we probably want to use aggregated trades because we care about the size/frequency of the aggregate order and not how it ended up impacting the book.

We may have situations where both are relevant, such as estimating the probability of getting filled at a level of the orderbook. We certainly will need to look at the individual trades to determine which levels of the book get impacted, but on the same note, we need to ensure that our simulation understands that all these orders arrive at the exact same time. If our model calculates that 10 of the individual orders from an aggregate order would have filled us, and we then make a hedge trade because of those fills - we can’t assume that liquidity can be re-used.

***Wash Trading & Fake Liquidity:***

Often, cryptocurrency exchanges add fake trades to the trade feed and try to prop up their volume statistics. When you actually go to quote in those books, you will find that these trades match at prices worse than yours - even if you are the best level in the book, all without you getting a single fill. Why is this? Fake flow. That’s why. It can lead to drastic overestimations of your ability to get filled in a book or the ability to slowly execute size through the book without being detected/moving the market. On the liquidity side, dirty exchanges will have fake quotes that get pulled immediately after one of them gets hit. You can still hit them, but the second you do, they’ll all disappear. Not all liquidity you see is really there with the intent of liquidity provision - some is there to meet the terms of an agreement and will flash away the second you test it. With this sort of liquidity, it is best to hit it all in one go and skip the VWAP/TWAP algorithms - take it before you lose it.

***Adjusted Data:***

Especially with financial data, the data will be edited after it is released. Perhaps there was an error or, worse, an accounting fraud took place - so after the fact, the data was edited to reflect the true value of it. The asset’s price for that period will not reflect this, of course, nor would the information present to traders at the time. Thus, you end up with look-ahead bias as you algorithm prices in the accounting fraud/error before it is even made public.

This is not just the case with financial data; many datasets are adjusted later on for various reasons. It is critical to ensure that the data is as it was when it was released and not in its post-correction form.

***Differences Between Data Vendors:***

Alternative data frequently has issues where the measurements of one provider is drastically different from another. An example where I have seen this consistently is with on-chain / fundamental data for cryptocurrencies. Vendors will provide wildly different estimates of various metrics, such as global volume, with some filtering out wash volume and others including it.

There are ways to get around this, with the best being to use multiple sources and triangulate the correct value using three or more vendors.

Flow data commonly has this issue as well, where estimates of OTC flow will vary due to the inclusion of different markets or even a lack of care from the vendors themselves.
