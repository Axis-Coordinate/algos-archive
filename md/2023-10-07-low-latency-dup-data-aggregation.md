# Low Latency Dup. Data Aggregation

Source HTML: [`html/2023-10-07-low-latency-dup-data-aggregation.html`](../html/2023-10-07-low-latency-dup-data-aggregation.html)

# Low Latency Dup. Data Aggregation

| 항목 | 값 |
| --- | --- |
| 날짜 | 2023-10-07 |
| 접근 | 유료 |
| URL | https://www.algos.org/p/low-latency-dup-data-aggregation |
| 부제 | A key piece of technology for fast data & state synchronization |

---

[![New data aggregation and analysis tools aid crisis response | FedScoop](images/8fef03aae753.png)](images/82fbe5330c7e.png)

#### Introduction

---

This is probably a departure from my usual discussion of research, but I thought I’d talk a bit about trading systems. Even on the research side, you’ll end up coding strategies, and having the skill set to implement them is immensely valuable.

My view remains that research is a better place to focus than developer skills. Any researcher can hire a developer to implement their strategies, but that dynamic does not apply in reverse. People coming from the developer side often focus too much on developer skills instead of understanding trading strategies & systems (the kind you only get from actually implementing them).

Despite it being my recommendation that developers focus more on trading strategies than tech understanding (this is probably overly general and only a reflection of the average quantitative developer putting too much focus there - so take it with a grain of salt), I think that for researchers being able to implement your own strategies, even if that is a dirty Python implementation, is hugely valuable. Time to production is key - get it done fast.

I doubt many would disagree with the view that the OMS (Order Management System) is by far the most complicated part of any trading system, but state synchronization & data aggregation probably come in for a close second place.

#### Order Management System

---

This article is not about OMS's, but I will link below to an article I think does the topic justice:

https://www.linkedin.com/pulse/how-do-i-design-high-frequency-trading-systems-its-part-silahian-1/

You’ll also find a lot of value in Professional Automated Trading. A great textbook for these topics.

#### Duplicate Data Feeds

---

I don’t think this component of a digital asset trading system has a name because it is very specific to digital assets and the way that latency gets optimized.

The key idea here is that we will be streaming a LOT of duplicate data. We want to present these updates to the trading algorithm without making it deal with many, many formats and often stale information.

We may have multiple of the same stream, or we may be streaming data that is, in theory (but not practice), redundant to data from other streams.

Using Binance as an example, we already have multiple endpoints for REST requests:

- https://api.binance.com
- https://api-gcp.binance.com
- https://api1.binance.com
- https://api2.binance.com
- https://api3.binance.com
- https://api4.binance.com

We also have many endpoints for WebSockets, and on top of this, these are only the ones that Binance tells you about. There are VIP endpoints only available to certain clients.

Binance (futures) sits in the C2 AZ in Tokyo for AWS, but on rare occasions, Singapore may be faster than locating in Tokyo. You can get tail latency scenarios where it is faster to get data from Singapore forwarded because of less traffic on the wire, even if the distance is longer.

You’re probably thinking, wow, we’ve already got tons of symbols, endpoints, and even AWS availability zones, but this is only the start.

You have multiple streams, and for the best latency, you will need to stream all of them:

1. 100ms Partial Book Depth Stream
2. 1000ms Partial Book Depth Stream
3. Quotes / Ticker
4. Trades
5. Orderbook Deltas
6. Private Fill Streams

But shouldn’t I be able to get all the information in the 1000ms Partial Book Depth Stream using the 100ms version? In theory, yes. In practice, no. If the streams are misaligned, then we may get the 100ms update, then 50ms later the 1000ms update, and then another 50ms later the next 100ms update. For many exchanges, this is a regular occurrence.

These are the spot streams. For futures, we have even more streams (100ms, 250ms, 500ms, …)!

In addition, we may have many ways to stream this data. Some exchanges have WebSockets but also offer FIX and/or gRPC. IPv4 or IPv6? Is there an older version of the API still able to be used that is faster on occasion?

All of this ends up with you needing to stream a lot of data.

#### Getting Data First

---

In general, the order in which data gets sent out goes as follows:

1. Trade occurs.
2. Private feeds are sent first.
3. Trades sent out.
4. Orderbook deltas / updates sent out.
5. Quotes sent out.
6. Orderbook snapshots sent out.

Somewhere in here, the database gets updated (which is our update time for REST calls if we use proxies and spam REST calls), and this is typically quite slow, but in extreme times (when the latency matters), we, on occasion, see this become faster so it is sometimes the case that people will spam (although rare).

Thus, we have our actual orderbook that gets sent out and our “shadow” orderbook, which we infer from things like quote updates and trades. If our fastest feed for full orderbooks is 100ms (and there is no real-time book updates feed - some exchanges are like this), then we have a 100ms period where we have to imply the state of the orderbook in between.

Even then, private fill updates & trades are usually sent before the actual book update gets sent out, so it is very common to get these first. In fact, there is an advantage to having orders in the book on exchanges because if you get hit by a large order, you know before most others to move your orders on other exchanges before that order impact propagates.

Finally, we may want multiple of the same feed, and every X seconds, we drop the feed that was the last to receive data the most times. The redundancy helps us avoid jitter on the wire, but we also rise to the top of the list. The list we are talking about is the list of where the data gets sent out to OR because most exchanges are distributed in the cloud for crypto; we are simply getting our data from a faster node with less traffic.

#### Sweeping This Under The Rug

---

We’ve talked a lot about how data servers can get data faster without needing to find unique tricks (hard to find, and nobody will tell you them, including me), but now it’s time to talk about how this relates to trading systems.

Depending on your need for latency and willingness to pay for cloud compute, you’ll usually pick between a dedicated data server or having this be part of the client. Either way, we will hide the fact that there are maybe 20 feeds giving us information for one ticker. Our algorithm only needs to be told the first time we get the data. If we have 3 of the same feed and they all give the same piece of data, we only need to pass on the first one to arrive. The rest can be discarded.

For strategies that have a lot of streams to the point where this requires all of the compute of a beefy instance, then you may see one central data server that takes in all our data and then spits out the first message to the strategies. These are right next to each other, and we can typically get this time to pass the data down to 150us - not too bad and probably worth it.

Not all strategies need this level of latency engineering and may instead opt to have a client that streams the orderbook, but really, behind the scenes, we have 10 streams which are then aggregated.

I have covered a LOT of ways to lower data feed latency, but doing all of this is usually seen as overkill. Most will do what makes sense for them and pick the tricks/endpoints/feeds that deliver the most value for the increase in compute / effort to implement (after a bit of testing).

#### State Synchronization

---

The general theme of this article is about how handling duplicate information is critical to engineering a low-latency digital asset trading system, but latency aside, it is still very important for tracking our state.

If we get an update for our orders and then set our active orders to the value of this update, but have recently sent out an order - we can end up with an incorrect tracking of our data.

Managing pending orders and locking the buy/sell side of a book OR handling errors for cases where we send a new order, then immediately decide to cancel it (for whatever reason), and this leads to an error because the cancel arrives before the new order.

Jitter and randomness on the wire generally can cause problems because the exchange offers no guarantee that a message sent before another will arrive first. We must track what orders are pending and when events occurred (us sending it, exchange recording the event, exchange sending it out, and us receiving it) or risk having an error-prone system that cannot properly track its inventory.

Finally, we must decide whether we want a slower but error-free system or a faster but error-prone system (errors can be handled, but this adds to the system complexity and may be worth skipping for certain strategies that aren’t massively latency concerned).

The example of our cancel arriving prior to our new order arriving can be prevented by queuing our cancel until we receive confirmation on the status of the new order. This adds latency, however.

We may also have latency optimizations that we KNOW will cause errors. If we have an order we desperately want to cancel, we may consider it worth spending our rate limit and sending 5 cancels in parallel. 1 will get there first and cancel the order, and the other 4 will produce errors that need handling.

This is a way to reduce our latency since there is a degree of randomness to our latency, so spamming tons of orders (and making sure we have idempotency) can reduce latency. This will be at the cost of our rate limit being used up quickly, so for latency-concerned firms, the rate limit may be a point to press exchange reps for.

#### Final Remarks

---

I hope that I provide an overview of the tooling for designing trading systems and improving latency, but I want to remind readers that doing all of this to reduce latency is probably a bit extreme, and someone who has found a unique trick will likely end up being faster than you still. You’ll still be faster than 99% of people, but latency is very much a winner-takes-all game, and these techniques are somewhat known (or at least they are now).

That said, there are a ton of trades where you’ll have a latency need and are in enough of a niche where there are only a few competitors. Even in general, there is still some benefit to not being an idiot on latency. I have laid out a lot of useful information; it’s now up to each reader to decide the extent they consider it worth utilizing for their strategy.
