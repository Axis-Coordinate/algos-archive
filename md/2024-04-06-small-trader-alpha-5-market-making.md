# Small Trader Alpha #5 - Market Making in Binary Markets

Source HTML: [`html/2024-04-06-small-trader-alpha-5-market-making.html`](../html/2024-04-06-small-trader-alpha-5-market-making.html)

# Small Trader Alpha #5 - Market Making in Binary Markets

| 항목 | 값 |
| --- | --- |
| 날짜 | 2024-04-06 |
| 접근 | 유료 |
| URL | https://www.algos.org/p/small-trader-alpha-5-market-making |
| 부제 | How to use existing option markets to price binary options for easy market making. |

---

[![](images/26f3b7edad44.webp)](images/2d8d69ebe43a.webp)

#### Introduction

---

Small trader alpha is a series of articles I write about real trades that are actually profitable at present. They exist because they are only available to small traders due to limited capacity. This doesn’t mean they should be shunned. It’s often the case that those who care to exploit it don’t know how and those who know how don’t care because they seek larger opportunities. Hence we bridge this gap in the small trader alpha series. With that said, on with the article:

A very interesting trade exists at the moment regarding market making event prediction markets for financial assets.

It’s not often where we get given a really great fair value model for free, but this is one of those examples.

Sites like Kalshi offer the ability to bet on yes/no outcome questions. This effectively ends up creating binary option markets when they start asking questions like:

“Will the S&P500 end the year over 4000?”

The flow on these bets are almost exclusively retail, but more importantly we can actually use option prices for the real tradfi markets on these assets to figure out the implied probability of outcomes.

In this article, we break down this opportunity and all of the components that would go into taking advantage of it from start to finish.

We include code and practical details of how to implement this. To my understanding there is a fairly decent amount of flow (if you’re a small trader or team, maybe not for bigger shops), lack of competition, and attractive incentives from exchanges.

There’s also a very good arbitrage in addition to all this that I’ll dive into.

#### Index

---

1. Introduction
2. Index
3. Understanding Binary Options and Their Relationship with Normal Options
4. Black-Scholes Model Overview
5. Pricing Binary Options Using the Black-Scholes Framework
6. Mathematical Derivation and Code Implementation
7. Interpolation of Strikes and Expiries
8. Sourcing Data
9. Cross-Venue Arbitrage
10. Managing Risk
11. Conclusion

#### Understanding Binary Options and Their Relationship with Normal Options

---

Binary options, often referred to as “all-or-nothing options,” pay off either a fixed amount or nothing at all depending on whether a certain condition is met, such as an asset price being above or below a specific level at expiration. This simplicity makes binary options akin to bets on the future price of an asset.

In contrast, normal options (such as European call or put options) give the buyer the right, but not the obligation, to buy or sell an underlying asset at a specified strike price on the option’s expiration date. Their value is influenced by factors like the underlying asset’s current price, the strike price, time until expiration, and volatility.

The connection between binary options and standard options lies in the concept of expiration and the conditions under which they become profitable. We can leverage the mathematical framework used for pricing standard options, particularly the Black-Scholes model, to derive a pricing model for binary options.

#### Black-Scholes Model Overview

---

The Black-Scholes model provides a theoretical estimate for the price of European-style options. The formula for a call option is given by:

C(S,t)=S0N(d1)−Xe−rTN(d2)

Where:

- C(S, t): Price of the call option
- S\_0: Current price of the underlying asset
- X: Strike price
- T: Time until expiration
- r: Risk-free interest rate
- N(d): Cumulative distribution function of the standard normal distribution
- σ: Volatility of the underlying assets returns
- d\_1:

d1=ln⁡(S0/X)+(r+σ2/2)TσT

- d\_2:

d2=d1−σT

#### Pricing Binary Options Using the Black-Scholes Framework

---

For binary options, the payout is fixed and occurs if the underlying asset is above (for a call) or below (for a put) the strike price at expiration. Therefore, we’re interested in the probability that the option expires in-the-money (ITM), which relates to in the Black-Scholes model.

**Binary Call Option Pricing:**

A binary call option pays out one unit of cash if the condition is met at expiration (i.e., the asset price is above the strike price). The price of a binary call option can be represented as:

Cbinary(S,t)=e−rTN(d2)

where d\_2 is defined as before, and e^-rT discounts the payoff back to the present value.

**Binary Put Option Pricing:**

Similarly, a binary put option pays out if the asset price is below the strike price at expiration. Its price can be represented as:

Pbinary(S,t)=e−rTN(−d2)

**Converting Implied Volatility to Binary Option Pricing:**

Implied volatility is a critical component in the pricing models. It represents the market’s view on the volatility of the underlying asset’s returns and is derived from the market prices of standard options.

To price a binary option using implied volatility from a standard option at the same strike, you first extract from the standard option’s market price using an iterative method (like the Newton-Raphson method) and then substitute into the binary option pricing formula.

**Code for Finding Implied Volatility:**

Here’s a quick example of how to find implied volatility from option prices using Newton-Raphson. It’s best to use py\_vollib\_vectorized in my experience since that’s faster, and more robust. The library in question has implementation guidance in the documentation online and uses the “Let’s be Rational” method for solving.

```
import numpy as np
from scipy.stats import norm

# Black-Scholes price of a European call option
def bs_call_price(S, K, T, r, sigma):
    d1 = (np.log(S / K) + (r + 0.5 * sigma**2) * T) / (sigma * np.sqrt(T))
    d2 = d1 - sigma * np.sqrt(T)
    return S * norm.cdf(d1) - K * np.exp(-r * T) * norm.cdf(d2)

# Vega of the call option (derivative of price with respect to volatility)
def bs_vega(S, K, T, r, sigma):
    d1 = (np.log(S / K) + (r + 0.5 * sigma**2) * T) / (sigma * np.sqrt(T))
    return S * np.sqrt(T) * norm.pdf(d1)

# Newton-Raphson method for finding implied volatility
def find_implied_volatility(S, K, T, r, market_price, sigma_estimate=0.2, tolerance=1e-5, max_iterations=100):
    sigma = sigma_estimate
    for i in range(max_iterations):
        price = bs_call_price(S, K, T, r, sigma)
        vega = bs_vega(S, K, T, r, sigma)

        # Avoid division by very small numbers
        if abs(vega) < 1e-10:
            break

        price_difference = market_price - price
        sigma += price_difference / vega  # Newton-Raphson update

        if abs(price_difference) < tolerance:
            return sigma

    # If the loop completes without finding a root, raise an exception
    raise ValueError("Implied volatility not found after maximum number of iterations")

# Example parameters
S = 100  # Underlying asset price
K = 100  # Strike price
T = 1    # Time to expiration in years
r = 0.05  # Risk-free interest rate
market_price = 10  # Observed market price of the option

# Estimate the implied volatility
implied_volatility = find_implied_volatility(S, K, T, r, market_price)
print(f'Implied Volatility: {implied_volatility}')
```

#### Mathematical Derivations and Code Implementation

---

Now, let’s derive d\_2 and implement the binary option pricing formula in code:

Given:

d1=ln⁡(S0/X)+(r+σ2/2)TσT

We know:

d2=d1−σT

Substituting d\_1 gives:

d2=ln⁡(S0/X)+(r−σ2/2)TσT

This d\_2 is then used in the binary option pricing formulas mentioned earlier.

**Code for Binary Option Pricing:**

```
import numpy as np
from scipy.stats import norm

def binary_option_price(S, X, T, r, sigma, option_type='call'):
    d1 = (np.log(S / X) + (r + 0.5 * sigma**2) * T) / (sigma * np.sqrt(T))
    d2 = d1 - sigma * np.sqrt(T)

    if option_type == 'call':
        price = np.exp(-r * T) * norm.cdf(d2)
    else:  # put option
        price = np.exp(-r * T) * norm.cdf(-d2)

    return price

# Example parameters
S = 100      # Underlying asset price
X = 100      # Strike price
T = 1        # Time to expiration in years
r = 0.05     # Risk-free interest rate
sigma = 0.2  # Implied volatility

# Calculate binary call and put prices
binary_call_price = binary_option_price(S, X, T, r, sigma, 'call')
binary_put_price = binary_option_price(S, X, T, r, sigma, 'put')

print(f'Binary Call Option Price: {binary_call_price}')
print(f'Binary Put Option Price: {binary_put_price}')
```

#### Interpolation of Strikes and Expiries

---

It won’t always be that the strike prices and expiration dates on the event betting platforms line up with real markets.

It’s a waste not to quote these misaligned options, especially given that we want to maximise capacity, so we interpolate the values for implied volatility to solve this.

To avoid repeating myself, I’ll link my article below on exactly how this is done which goes into tons of detail:

[![Finding Fair Value [CODE INSIDE]](images/ea553a6c0209.png)Finding Fair Value [CODE INSIDE][Quant Arb](<https://substack.com/profile/101799233-quant-arb>)·March 10, 2024[Read full story](<https://www.algos.org/p/finding-fair-value-code-inside>)](https://www.algos.org/p/finding-fair-value-code-inside)

#### Sourcing Data

---

Event betting platforms will provide an API so the data there is pretty straightforward and no-cost.

You will, however, need a live options pricing feed and a live feed of underlying prices.

It’s much easier to get a fast feed of the underlying than of option prices so if you’d like to save money I recommend using a slow feed for option pricing then shifting that curve along recent price updates using the tangent method of the Wing model outlined in the finding FV article. For most though, this is overkill and more worth it to fork over the money.

For a data sourcing guide, see prior work:

[![Data Sourcing - The Guide](images/8fabc6e1471b.png)Data Sourcing - The Guide[Quant Arb](<https://substack.com/profile/101799233-quant-arb>)·January 2, 2024[Read full story](<https://www.algos.org/p/data-sourcing-the-guide>)](https://www.algos.org/p/data-sourcing-the-guide)

#### Cross-Venue Arbitrage

---

In addition to this opportunity, there’s massive arbitrages between venues - not necessarily in the financial bets we describe, they extend well beyond this to bets such as political outcomes and many more. Of course these arbitrages require no pricing model short of A-B-fees (if they’re settled the same way and expire at the same time which you need to watch out for).

There’s not much to say here, it’s clear cut how you calculate these, determine the APR, and if you need extra juice maker into it.

#### Managing Risk

---

You can still use vega and delta as with any option to know how each option contributes to your risk (roughly, it’s not a direct translation).

You also should have individual position limits given how a slight change can have huge impacts, giving a quite non linear risk profile at the local range.

Look at the Deribit margin system documentation for a good example of how to calculate scenario risk. Shocking your book across scenarios like this is the best way to ensure you are within risk bounds. Beyond that you may want to decide for yourself which of their contingencies on top of this apply.

#### Conclusion

---

We’ve shown an example of a small trader alpha which should provide a smooth introduction to market making where the FV problem is directly solve-able with minimal guesswork (tradfi markets are obviously the best predictor).

Thanks for reading, feel free to DM any questions on Twitter.
