# Hedging Crypto Exchange Risk - How/Where

Source HTML: [`html/2023-03-08-hedging-crypto-exchange-risk-howwhere.html`](../html/2023-03-08-hedging-crypto-exchange-risk-howwhere.html)

# Hedging Crypto Exchange Risk - How/Where

| 항목 | 값 |
| --- | --- |
| 날짜 | 2023-03-08 |
| 접근 | 무료 |
| URL | https://www.algos.org/p/hedging-crypto-exchange-risk-howwhere |
| 부제 | An inside view of crypto exchange insurance and effective hedges against counterparty risk |

---

#### Introduction

---

I recently received quite a few questions when I brought up crypto exchange insurance in a thread defending Bybit / giving my view on the recent FUD so I thought I would write an article that covers this topic. We will cover 4 sources that can be used to hedge risk, and touch on the topic of proxy assets before giving some final remarks.

Whether you just want to see the rates so you can judge whether your assets are safe or you are actually interested in hedging this risk out, I’ve got you covered. None of this constitutes financial advice and the majority of this article is freestyled with some googling in between. Unlike my usual content, I wouldn’t classify this article as a research article so please run your own checks before picking counterparties.

Insurance in digital asset markets, whether that is for the exchange or protocols themselves is a market with high premiums. There is a lack of data and these events have long tails. This combination produces a high risk-premium for the insurers so hedging risk can be an expensive endevour. The market for cybersecurity insurance is a great example of a mature insurance market with large risk premiums due to tail risk. We will explore how different sources compare in terms of liquidity, cost, accessibility, and flexibility.

Quant’s Substack is a reader-supported publication. To receive new posts and encourage future shitposting, consider becoming a subscriber.

#### Traditional Insurance

---

This is not something that is available to retail or a source where you’ll be able to regularly check the market rate, but it is a market that can provide deep liquidity at a reasonable price.

It is recommended that a broker is used to find the right counterparty as deals like this are almost always OTC. There are multiple insurance firms in the space that are either traditional insurers or have emerged to specifically target the digital assets industry.

If you are looking to hedge large sizes, but want to retain custody of your assets, unlike with a prime broker, this is going to be your best bet. This route is best used for hedging purposes instead of speculating against an exchange. Counterparties will not be thrilled at the idea of potentially getting adverse filled, especially in an industry like digital assets where insider information is common.

As a result of it being OTC, there is no ability to get a quote (without considerable effort) just to check how the market is pricing the risk. This is definitely not an option retail would be able to use and appetite for this risk from insurers has decreased post-FTX.

#### Prime Brokers

---

Prime brokers (PBs) offer many services to clients, and one of the major PBs, Hidden Road Partners (HRP), charges an insurance fee that covers you against exchange default events. In equities markets, this is usually provided without extra costs and your only counterparty is the PB, but in crypto, there is a lot more risk and firms tend to be less established. Thus, there is a higher cost of capital for funding the necessary reserves.

HRP actually gives counterparties a loan for their trading capital so you actually retain custody of the collateral the entire time. With HRP you don’t even have exposure to the PB.

I usually recommend FalconX (although HRP is great), so if you are dealing with other PBs you’ll have to ask them about their ability to accommodate such needs.

Once again, this is not available to retail and the rates have to be requested (although they are a fair bit easier to get a hold of). HRP’s rates are pretty attractive in my view, last time I checked they were at 5% for major exchanges (9% for Huobi due to extra risk). I expect that this is to increase client business by making it more affordable.

NOTE: These rates are only from a month or two ago, they may have changed and will of course change in the future.

#### Speculation Markets

---

Speculation markets offer a way to hedge exchange risk that is far more accessible to retail users than the previously mentioned options. I will give some caution that I am not the most familiar here and so I’ll leave it up to the reader to find the right marketplace. These are markets where people can bet on the outcomes of events like elections. Often, these markets can be used to speculate on the probability that an exchange goes bust within a specified timeframe.

These markets are highly illiquid and are really only ideal for smaller players. The premium may not be in line with what the true risk is, but a large change in premium can typically be viewed as a poor sign. I will note that because these markets are illiquid it is wise to ensure that this change was not just a liquidity shock when forming a view.

Liquidity picked up massively for markets like this around the FTX saga as demand for exchange risk hedges shot through the roof. Thus, they tend to become a lot more feasible for firms that need to hedge in size when shit hits the fan.

#### CDS Markets

---

Credit Default Swaps (CDS) markets have traditionally been used in equities markets to hedge these sorts of risks. I’ve not explored this space too much so can’t give a lot of content here, but there have been OTC deals in the past where CDSs were sold against FTX.

There are USDT depeg CDSs and I do know of a startup that has created a CDS to protect against smart contracts failing. These examples aren’t quite exchange risk hedging, but as depeg and smart contract failure risk markets expand, we will see exchange risk products also grow in size, popularity, and accessibility.

This is a market where you’ll have to do your own research on the counterparties in the space, but it is a very small market and OTC asf so not very easy to get info on.

#### Proxy Hedges

---

Proxy hedges are a term I’ve decided to make up whilst writing this thread, but effectively it is any dirty method of hedging out counterparty / event risk in products that were not originally made for this purpose.

Starting with USDT depeg risk, a synthetic short USDT/USD position can be created using a mix of USDT and USD quoted BTC futures (or whatever is cheapest). The difference in funding rates will be the cost you pay out. This is an example of using a spread between two assets to hedge risk.

During the FTX saga, we saw another proxy asset emerge - TRX. At this point, the collapse of FTX was a given so it existed more as a proxy for deposit salvage value. Due to the deal with the TRON foundation where users could withdraw their TRX from FTX over to Huobi, TRX traded at a massive premium on FTX compared to other exchanges. This premium could be used to determine the market price for an FTX deposit. This market was heavily FUD driven / only really tradeable for non-speculators, i.e. only people pulling their assets so wasn’t always trading at fair value.

I’m sure there are many other examples of how hedges like this could be constructed, one idea off the top of my head is a short position on a safe exchange against another exchange’s token, hedged with a long basket of other exchange tokens. This would probably make the most sense as an “active hedge” where you only put it on when there is reason to worry. Otherwise, noise will drown out your hedge.

#### Conclusion

---

For retail, the best approach is always to exercise caution and keep your primary holdings in a secure wallet. Larger firms have a lot more room to hedge this risk out and be informed about the fair value of such hedges, but there are still options for retail - mainly speculation markets or proxy hedges.

I personally prefer my USDT, much like SBF in his penthouse, pegged hard. This is an equally important risk that can actually be hedged using the synthetic spread approach mentioned before. A simple way to avoid this risk with prime brokers is to put your collateral in USD and borrow against it in USDT.

With the exception of PBs, where it is actually quite affordable, this sort of insurance commands a high premium. Searching around for the best source of protection is wise and will save you a fair bit of money if you decide to put on a hedge.

The best way to protect yourself, especially for retail, is to not hold assets that don’t need to be on exchange in your exchange wallet. It doesn’t take that long to transfer back on an exchange so you can rest assured that your assets are still very liquid.

For protocol insurance, I recommend Nexus Mutual, but you should examine all options yourself (and generally should do your own research for any of this).

Be very careful on the cybersecurity side when using off-exchange wallets as it is a serious risk (equal concern as exchange risk itself in my view). Hackers are insanely crafty, and whilst I’m not awfully qualified in this area I will advise that you use an intermediate wallet to transfer between your long-term holdings wallet and your exchange wallet.

If you enjoyed reading consider subscribing :)

Quant’s Substack is a reader-supported publication. To receive new posts and support my work, consider becoming a free or paid subscriber.
