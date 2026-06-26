# Data Sourcing - The Guide

Source HTML: [`html/2024-01-02-data-sourcing-the-guide.html`](../html/2024-01-02-data-sourcing-the-guide.html)

# Data Sourcing - The Guide

| 항목 | 값 |
| --- | --- |
| 날짜 | 2024-01-02 |
| 접근 | 무료 |
| URL | https://www.algos.org/p/data-sourcing-the-guide |
| 부제 | A guide to scraping / finding data for all your quant needs |

---

[![](images/a293883c391f.png)](images/71301956f23c.png)

#### Introduction

---

This article has a bit of a focus on digital assets as this is the industry I work in and thus can provide a very comprehensive overview of, but I also try to cover as much alternative options as possible - I am quite the data hoarder afterall.

We place our data into 4 tiers of pricing:

1. Free
2. Dirt Cheap
3. Retail Friendly
4. Institutional

After purchasing a subscription for backtesting / research purposes, it is wise to be proactive about scraping. It is often the case that beginners simply want this data for a project and not for continued use in production / rapidly changing research where up-to-date data is necessary. Thus, I recommend that you scrape as much data as possible before the subscription runs out.

If you enjoy this article or my tweets, feel free to become a free or paid subscriber. I put out long form content where I share research & knowledge derived from my experiences as a practitioner.

#### Index

---

1. Introduction
2. Index
3. Crypto HFT Data
4. Crypto Alternative Data
5. Tiingo [Dirt Cheap]
6. Polygon [Retail Friendly]
7. WRDS [Academic Only]
8. News & Sentiment Providers [Institutional]
9. IBKR [Dirt Cheap]
10. QuantConnect [Dirt Cheap]
11. List of additional providers

#### Crypto HFT Data

---

Digital assets have one great element of being very low barrier of entry. This is similarly the case for the data as well. You can get any live data you want directly from the exchange, free of charge. Most exchanges will have access to historical data, but will not provide extremely granular data such as orderbooks. Binance has a whitelist program where if you have high enough VIP they’ll let you download it, but it’s slow and frankly a paid option is still better.

Hence, there are two main providers. Tardis (tardis.dev) and Kaiko (kaiko.com). Tardis is simply better, but Kaiko is simply broader. If Tardis has it, use Tardis, if Tardis doesn’t use Kaiko.

Tardis has better support, better quality of data, better tooling around that data, and all around I recommend them more (they’ve always been my preferred route).

That said, Kaiko is a great option - Tardis is just really hard to compete with. Kaiko finds it’s niche in having a much broader range of data. If Tardis doesn’t have it - Kaiko will.

As an extra note, if you want to scrape free data, use CCXT as it will make it much faster / normalized and if you want a live feed of normalized quotes / orderbooks / generally normalized HFT data - then the Tardis machine is a free and high quality option (does not require a Tardis subscription to use).

Kaiko is more expensive and aimed only at institutions, whereas Tardis offers packages for academic, solo, and institutional players.

If Tardis is still too expensive, then crypto lake (crypto-lake.com) offers data at a much cheaper rate, but with less coverage.

#### Crypto Alternative Data

---

Glassnode is a very affordable option, however, there is no longer an API for the $40 a month option. This of course doesn’t pose much of an issue if you simply scrape the graphs on the website:

For crypto fundamentals otherwise, I recommend Santiment.

#### Tiingo

---

Tiingo is just great. There’s not much more to say. Easily the best value for money. $10 a month gets you access to most asset classes OHLCV data, fundamental data, and news data.

#### Polygon

---

Polygon is like Tiingo but more expensive and with more data. It’ll have the data that’s a lot larger like options / equities HFT data (historically), but it’ll cost more. Luckily, still within retail ranges, but this is a fair bit more - still quite a great deal.

#### WRDS

---

WRDS - Wharton Research Data Services gives an insane amount of data, and of course it is free, but only if you have access (through a university). I have sure scraped my fair share of this website using keys from friends - it’s a treasure trove of data.

Sadly, the data is not going to be quite the quality needed for accurate HFT research, such as the TAQ dataset, which isn’t a high quality as the ones institutions will have access to. You can still get a lot done with it, but there is ample evidence in the literature of people underestimating liquidity when compared to those who used better, institutional, datasets.

The range of alternative data sources was very interesting though. Millions of employee profiles scraped off LinkedIn, accounting filings (GBs), all sorts.

#### News & Sentiment Providers

---

Tiingo has a very well priced feed, but it is very basic.

For institutional players, Ravenpack & Alexandria Research are two of the main options.

Each will set you back around $60,000 for the crypto news subscription or $100,000 for equities news subscriptions.

These analyze any sorts of media, label them as to what they relate to, give them sentiment scores, and pass the data on to you with very low latency.

The equities datasets used to be very profitable a while back, but not the case anymore from my current understanding.

#### IBKR

---

IBKR is extremely cheap as a retail user, only $5-10 a month for certain subscriptions, but the main downside is that it takes ages to scrape.

Sure you can get an expensive options data subscription for $10 a month, but it’ll take you a whole month just to scrape it.

The API is insanely slow and the rate limits mean that you will probably need to set up an AWS server just to scrape this from IBKR. Maybe things have changed since I last scraped the website, but I still hear complaints so I doubt it.

That said, if you’ve got time to scrape and money to save - IBKR is THE place to go.

#### QuantConnect

---

QuantConnect is a unique option because whilst it has a lot of interesting alternative datasets / generally a very high quality of data, and at low cost as well, it does come with the caveat that you can’t take it outside of the platform.

You will need to deploy and research all your strategies in their platform.

That said, QuantConnect is not a bad place to deploy strategies as a retail trader, and it makes the process much faster - it isn’t great for research though and you’ll certainly have a lot of hindrance there because of it.

#### List of Additional Providers

---

Here is a quick list of sources I have never used before, but may come in handy for the avid data searcher.

<https://firstratedata.com/>

<https://tickstory.com/download-tickstory/>

<https://quantpedia.com/links-tools/?category=historical->

[www.spikeet.com](http://www.spikeet.com/)

[www.marketstack.com](http://www.marketstack.com/)

[www.nanex.net](http://www.nanex.net/)

[www.intrinio.com](http://www.intrinio.com/)

[www.fmpcloud.io](http://www.fmpcloud.io/)

[www.algoseek.com](http://www.algoseek.com/)

[www.alphavantage.co](http://www.alphavantage.co/)

If you are a hedge fund / well funded institution, here is a quick list of the expensive, but institutional quality options (outside of crypto, Tardis/Kaiko are the best options if in crypto as an insto):

1. Bloomberg
2. Refinitiv
3. MorningStar
4. Reuters
5. FactSet
6. FIS

Databento is also worth checking out :)
