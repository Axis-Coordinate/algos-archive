# Finding Fair Value [CODE INSIDE]

Source HTML: [`html/2024-03-10-finding-fair-value-code-inside.html`](../html/2024-03-10-finding-fair-value-code-inside.html)

# Finding Fair Value [CODE INSIDE]

| 항목 | 값 |
| --- | --- |
| 날짜 | 2024-03-10 |
| 접근 | 유료 |
| URL | https://www.algos.org/p/finding-fair-value-code-inside |
| 부제 | Regressions, Splines, and Marks |

---

[![](images/89361744ad04.png)](images/835036234a7b.png)

#### Introduction

---

Many people try to begin market making and struggle to understand what is meant by fair value or the process of establishing it. Most people think that this comes out of some model that is entirely new, but that couldn’t be further from the truth.

We’ll cover how to establish where the market views the fair value to be in spot, futures, and options markets and then move on to how to convert this into our own estimate of fair value.

Succinctly there are 4 main problems when it comes to market making:

1. What is my fair value?
2. What is my spread?
3. What is my skew?
4. What is my size?

Your fair value is what you quote around. The spread is the average distance between your bid/ask quotes and your fair value. The skew is the difference in distance from a fair value between the bid and ask, respectively. Of course, size is how much you quote.

Size is sometimes negligible; perhaps if you are quoting just one symbol or a small universe, then you weigh it based on some established share of your account, but when there are a lot of symbols, such as with shitcoins or options, you run into the issue of how to allocate your capital between different books.

For now, we will simply tackle the first problem and focus on showing how this problem is approached and solved. You still need to do a lot of research independently, but this should give you the tooling.

#### Index

---

1. Introduction
2. Index
3. Fitting Fair Value
4. SVI Surface Parameterization
5. Regressions
6. Wing Model
7. Conclusion

#### Fitting Fair Value

---

When looking at any orderbook, we need to isolate one price as fair value. In most cases, this will be the mid-price or simply the middle of the orderbook, but in cases where we feel there is a heavy imbalance of liquidity, we can weigh by bid/ask sizes.

The question then arises as to how many levels should we include in our weighting. This is best put in the context of why we weigh this at all - the reason being to get an accurate idea of where the price will be in the future. If we feel that imbalance typically will push the price in one direction (which it does), then we should select the number of levels where there is maximal predictability and relevance.

This makes things very complicated, though, so it’s a very late-stage optimization. In 99% of cases, the mid-price is fine. I wouldn’t even bother with micro-price (refer to the Stoikov paper on this to understand micro-price).

Once we have done this, we can use it as our “dumb” fair value. We will improve upon it by shifting it, but we start from the market price as our basis vector. For futures and spot, we simply take the midprice or weighted-midprice and call it a day, but for options, we need to fit a smooth curve for our volatility smile and even a 3D surface to get our volatility surface, which we quote around.

#### SVI Surface Parameterizations

---

Splines are employed to find this smooth surface. Most large trading firms will have complicated models with tens of parameters, but for now, we will start with the most advanced model that exists in the public literature.

We start with SVI. This is a spline model. Effectively, we want to fit a smooth surface which ensures consistency across our quotes and implied volatilities so they all make sense pricing wise when put together. We achieve that via the SVI model.

wimpsvi=a+b⋅(ρ⋅(x−m)+(x−m)2+σ2)

##### Parameters

- `x` (float): Moneyness, typically defined as the log of the strike price divided by the spot price.
- `a` (float): Parameter `a`, represents the base level of the total implied variance across strikes.
- `b` (float): Parameter `b`, modulates the volatility skewness/smile.
- `sigma` (float): Parameter `sigma`, controls the convexity of the volatility smile.
- `rho` (float): Parameter `rho`, determines the skewness or the slope of the volatility smile. It is the correlation coefficient between the asset return and its volatility.
- `m` (float): Parameter `m`, signifies the mode or the peak of the smile curve.

We characterize the raw parameterization of the total implied variance (assuming some static maturity) in the above formula, with all values other than x estimated from the market (we know moneyness).

```
def calculate_total_implied_variance(moneyness: float,
                                     base_level: float,
                                     skew_modulation: float,
                                     smile_convexity: float,
                                     smile_slope: float,
                                     smile_peak: float) -> float:

    return base_level + skew_modulation * (smile_slope*(moneyness - smile_peak) + ((moneyness - smile_peak)**2 + smile_convexity**2)**0.5)

# Example: Calculate the total implied variance for given parameters
x = 0.05  # Example moneyness
a = 0.04  # Base level of total implied variance
b = 0.1   # Volatility skewness/smile modulation
sigma = 0.2  # Convexity control of the volatility smile
rho = -0.5  # Skewness/slope of the volatility smile
m = 0.01  # Peak of the smile curve

w_imp_svi = calculate_total_implied_variance(x, a, b, sigma, rho, m)
print(f"Total Implied Variance: {w_imp_svi}")
```

NOTE: For our market data, we need to calculate time to expiry as a fraction of a year, calculate log-moneyness, and remove OTM options.

In order to estimate the best parameters, we need to solve using ordinary least squares, minimizing the below function:

mina,b,σ,ρ,m∑i=1n(wraw(x,a,b,σ,ρ,m)−wmarket)2

```
from typing import List, Tuple
import pandas as pd
import numpy as np

def calculate_residuals(params: Tuple[float, float, float, float, float],
                        time_to_expiry: float, 
                        market_data: pd.DataFrame) -> np.ndarray:
    """
    Calculates the residuals between the market implied volatilities and the volatilities
    predicted by the Raw SVI model for a given time to expiry.

    This function is used within a least-squares optimization process to find the SVI
    parameters that best fit the market data for each time to maturity slice.

    Parameters:
    - params (Tuple[float, float, float, float, float]): A tuple containing the SVI parameters
      (a, b, sigma, rho, m) to be optimized.
    - time_to_expiry (float): The specific time to expiry (in years) for which the residuals
      are being calculated.
    - market_data (pd.DataFrame): A DataFrame containing market data, which must include columns
      for 'time_to_expiry_yrs' (time to expiry in years), 'log_moneyness' (the log of moneyness),
      and 'mid_iv' (the market's median implied volatility).

    Returns:
    - np.ndarray: An array of residuals between the market and model implied volatilities
      for the specified time to expiry.
    """
    # Filter market data for the specific time to expiry
    specific_expiry_data = market_data[market_data['time_to_expiry_yrs'] == time_to_expiry]

    # Calculate the total implied variance using the Raw SVI model for filtered data
    w_svi = np.array([raw_svi(x, *params) for x in specific_expiry_data['log_moneyness']])

    # Extract the actual market implied volatilities
    iv_actual = specific_expiry_data['mid_iv'].values

    # Calculate residuals between market implied volatilities and model predictions
    residuals = iv_actual - np.sqrt(w_svi / time_to_expiry)

    return residuals
```

We then optimize using the below code:

```
from scipy.optimize import least_squares
import pandas as pd
from typing import List

def optimize_svi_parameters_for_all_expiries(market_data: pd.DataFrame) -> List:
    """
    Iterates over all unique expiry times in the market data, optimizes the SVI model
    parameters for each expiry using the least-squares method, and prints the outcome
    of each optimization attempt.

    Parameters:
    - market_data (pd.DataFrame): The market data DataFrame, which must include a column
      named 'time_to_expiry_yrs' indicating the time to expiry in years for each data point.

    Returns:
    - List: A list of optimization result objects from the `least_squares` method for each
      unique time to expiry in the market data.
    """
    results = []
    unique_expiries = market_data['time_to_expiry_yrs'].unique()

    for t_dte in unique_expiries:
        initial_guess = [0.0, 0.0, 0.0, 0.0, 0.0]
        result = least_squares(calculate_residuals, initial_guess, args=(t_dte, market_data), max_nfev=1000)
        results.append(result)

        if result.success:
            print(f'Ran for t_dte = {t_dte:.4f}: SUCCESS')
        else:
            print(f'Ran for t_dte = {t_dte:.4f}: FAILED')
        print('----------------------')

    return results
```

Finally, to get our parameters matrix:

```
import pandas as pd
from typing import List
from scipy.optimize import OptimizeResult

def create_parameters_matrix(market_data: pd.DataFrame, optimization_results: List[OptimizeResult]) -> pd.DataFrame:
    """
    Creates a matrix (DataFrame) of the optimized SVI parameters for each unique time to expiry,
    based on the results of the least-squares optimization.

    Parameters:
    - market_data (pd.DataFrame): The market data DataFrame, which must include a column named
      'time_to_expiry_yrs' indicating the time to expiry in years for each data point.
    - optimization_results (List[OptimizeResult]): A list of optimization result objects from the
      `least_squares` method for each unique time to expiry.

    Returns:
    - pd.DataFrame: A DataFrame where each column represents a unique time to expiry, and each row
      represents one of the SVI parameters (`a`, `b`, `sigma`, `rho`, `m`). The cell values are the
      optimized parameters for the corresponding expiry.
    """
    # Creating a DataFrame with unique times to expiry as columns and SVI parameters as rows
    param_matrix = pd.DataFrame(
        columns=market_data['time_to_expiry_yrs'].unique(),
        index=['a', 'b', '\u03C3', '\u03c1', 'm'],  # Unicode for sigma and rho
    )

    # Filling the matrix with optimized parameters from the results
    for i, result in enumerate(optimization_results):
        time_to_expiry = market_data['time_to_expiry_yrs'].unique()[i]
        param_matrix[time_to_expiry] = result.x

    return param_matrix
```

Putting it all together:

[![](images/c339b4459ec3.png)](images/64b5acf34069.png)

[![](images/9c16bb6d3710.png)](images/6bff3b9a5328.png)Options data can be found easily with the help of my guide I wrote on data-sourcing :)

We then generate our 2D surface from this parameters matrix:

```
import numpy as np
import pandas as pd
from typing import Tuple, Dict

def generate_implied_volatility_surface(
    market_data: pd.DataFrame,
    param_matrix: pd.DataFrame,
    min_m: float = -2.25,
    max_m: float = 2.25
) -> Tuple[np.ndarray, Dict[float, np.ndarray]]:
    """
    Generates the implied volatility surface for different moneyness and expiry times using
    the Raw SVI model parameters obtained from optimization.

    Parameters:
    - market_data (pd.DataFrame): The market data DataFrame, which must include a column
      named 'time_to_expiry_yrs' indicating the time to expiry in years for each data point.
    - param_matrix (pd.DataFrame): A DataFrame containing the optimized SVI parameters for
      each unique time to expiry. Each column represents a unique time to expiry, and each row
      represents an SVI parameter (`a`, `b`, `sigma`, `rho`, `m`).
    - min_m (float, optional): The minimum moneyness to use for generating the surface. Defaults to -2.25.
    - max_m (float, optional): The maximum moneyness to use for generating the surface. Defaults to 2.25.

    Returns:
    - Tuple[np.ndarray, Dict[float, np.ndarray]]: A tuple where the first element is an array
      of moneyness values, and the second element is a dictionary mapping each time to expiry
      to an array of implied volatilities corresponding to the moneyness values.
    """
    # Generate moneyness values within specified range
    moneyness_values = np.linspace(min_m, max_m, num=100)
    implied_volatility_surface = {}

    # Iterate over each unique time to expiry to calculate implied volatilities
    for T in market_data['time_to_expiry_yrs'].unique():
        w_ssvi = [raw_svi(x, *param_matrix[T].values) for x in moneyness_values]
        implied_volatility_surface[T] = np.sqrt(np.array(w_ssvi) / T)

    return moneyness_values, implied_volatility_surface
```

[![](images/d60e7f5cc7df.png)](images/555f5af2455a.png)

We then take the above code and produce the plot below, which shows an smooth volatility curve for each expiry.

[![](images/75ea447ce305.png)](images/162d6813a992.png)

To finally get a 3D surface:

```
import numpy as np
from scipy.interpolate import pchip_interpolate

def interpolate_surface(x: np.ndarray, y: np.ndarray, z: np.ndarray, no_points: int) -> Tuple[np.ndarray, np.ndarray]:
    """
    Interpolates the implied volatility surface along one dimension.

    Parameters:
    - x (np.ndarray): The original moneyness values.
    - y (np.ndarray): The original time to expiry values.
    - z (np.ndarray): The original implied volatility values.
    - no_points (int): Number of points to interpolate over in the y-dimension.

    Returns:
    - Tuple[np.ndarray, np.ndarray]: A tuple containing the interpolated y values and
      the corresponding interpolated z values.
    """
    # Interpolate y values
    intr_y = np.linspace(min(y), max(y), no_points)
    intr_z = []

    # Interpolate z values for each moneyness point
    for idx, point in enumerate(x):
        temp = pchip_interpolate(y, z[:, idx], intr_y)
        intr_z.append(temp)

    return intr_y, np.array(intr_z).T


import matplotlib.pyplot as plt
from mpl_toolkits.mplot3d import Axes3D
from typing import Dict

def plot_surface(market: pd.DataFrame, m: np.ndarray, iv_ssvi: Dict[float, np.ndarray], 
                 interpolate: bool, no_points: int = 25) -> None:
    """
    Plots the BTC implied volatility surface, optionally with interpolation.

    Parameters:
    - market (pd.DataFrame): The market data DataFrame.
    - m (np.ndarray): The moneyness values.
    - iv_ssvi (Dict[float, np.ndarray]): A dictionary mapping expiry times to implied volatility arrays.
    - interpolate (bool): Whether to interpolate the surface or not.
    - no_points (int, optional): Number of points for interpolation. Default is 25.

    This function plots the implied volatility surface using a 3D plot, where the x-axis represents
    the log moneyness, the y-axis represents time to expiry in years, and the z-axis represents the
    implied volatility percentage.
    """
    fig = plt.figure(figsize=(12, 12))
    ax = fig.add_subplot(111, projection='3d')
    y = market['time_to_expiry_yrs'].unique()
    z = [iv_ssvi[T] for T in y]
    z = np.array(z)

    intr_y, intr_z = y, z
    if interpolate:
        intr_y, intr_z = interpolate_surface(m, y, z, no_points)

    X, Y = np.meshgrid(m, intr_y)
    Z = intr_z * 100  # Convert to percentage

    surf = ax.plot_surface(X, Y, Z, cmap="coolwarm")
    ax.set_xlabel('Log Moneyness [ln(S/k)]')
    ax.set_ylabel('Time to Expiry (yrs)')
    ax.set_zlabel('IV (%)')

    title = 'BTC Implied Volatility Surface (Interpolated)' if interpolate else 'BTC Implied Volatility Surface'
    ax.set_title(title)

    fig.colorbar(surf, shrink=0.5, aspect=5)

    plt.show()
```

[![](images/f23c5b0c6f6c.png)](images/ede864686954.png)

When we plot it with, and without interpolation, in the above two cells, we get these beautiful plots:

[![](images/d273871846b8.png)](images/b1b2d09fe7b7.png)

[![](images/21b25e7ebeef.png)](images/1d84a1180d71.png)

%matplotlib qt for the 3D interactive plots you can play around with ;)

For some further reading on this topic, here are a few papers:

![](images/8ff66707eb30.svg)

Dynamicsofssvi

345KB ∙ PDF file

[Download](https://www.algos.org/api/v1/file/939f19b7-6b0e-44a6-a6b3-265d59bd0314.pdf)

[Download](https://www.algos.org/api/v1/file/939f19b7-6b0e-44a6-a6b3-265d59bd0314.pdf)

![](images/8ff66707eb30.svg)

Essvi

500KB ∙ PDF file

[Download](https://www.algos.org/api/v1/file/df8f5030-97c1-426d-b86b-bf8c1fa92040.pdf)

[Download](https://www.algos.org/api/v1/file/df8f5030-97c1-426d-b86b-bf8c1fa92040.pdf)

![](images/8ff66707eb30.svg)

Calibratedssvimethod

3.52MB ∙ PDF file

[Download](https://www.algos.org/api/v1/file/36d4c043-0b9b-4612-87d5-7751b5a7114d.pdf)

[Download](https://www.algos.org/api/v1/file/36d4c043-0b9b-4612-87d5-7751b5a7114d.pdf)

![](images/8ff66707eb30.svg)

Svi

4.57MB ∙ PDF file

[Download](https://www.algos.org/api/v1/file/bda04b5c-4bf0-4820-86b9-b1ba53237d60.pdf)

[Download](https://www.algos.org/api/v1/file/bda04b5c-4bf0-4820-86b9-b1ba53237d60.pdf)

#### Regressions

---

From here, we turn to regressions to adjust our parameters. Say we want to adjust the overall implied volatility of our surface, well we would need to augment our model to incorporate a parameter that captures the overall implied volatility of our surface and regress changes in spot price or any variable we think might be good for predicting the overall implied volatility, against the parameter. We are simply re-fitting the model regularly and looking at changes in parameters we want to fit ahead of time against changes in variables we think predict this.

Sure, we can keep refitting, but we’ll always be one step behind everyone else. Isn’t it better to know that if the spot moves 4%, you know exactly how to adjust your surface? In many markets, being able to adjust faster (or more accurately) than others to the input variables that ultimately create the surface is what makes you money.

That’s why the private sector models are incredibly complicated and can have tens of parameters that govern things like detailed put and call curvature—and these many parameters for each expiry on each instrument.

Going back to spot/futures, the process for improving our estimate of fair value works the same, except we only have one parameter to forecast (our midprice) instead of needing to aggregate our fair value for many options into a set of parameters for our curve. If the price moves on a large, dominant exchange, how does that affect my price on a smaller exchange?

How does the size of that order change it? Many factors can be considered, and that’s why these models tend to get quite complicated where they are very simple regressions between basic features, but with so many of them they capture and incredibly complicated system of dynamics when put together.

#### Wing Model

---

We’ve found a nice surface to quote around, but we want something that can be intuitively played with. That’s where we get to the Orc Trader Wing Model. This is one of those models I discussed regarding private sector models.

The Orc Wing model, is a slightly older model that was used by Optiver and most major options market-making firms for over a decade. I’ve been able to get some Python code together to replicate this and show readers a glimpse into how this works - even despite the fact that it’s remained miles ahead of the academic literature to this day.

The Wing Model combines a piecewise quadratic equation with linear extensions. This model segments the volatility smile curve into six distinct sections, utilizing four key interval boundaries:

dc(1+dsm), dc, uc, and uc(1+usm)

We can see these 6 sections annotated clearly below:

[![波动率模型（1）Wing Model](images/9c571d577c9a.png)](images/60a261a9164a.png)

![](images/8ff66707eb30.svg)

Orcwingmodel Specs

81.8KB ∙ PDF file

[Download](https://www.algos.org/api/v1/file/a9d751e6-35a8-4ef9-af90-f566b91a1834.pdf)

[Download](https://www.algos.org/api/v1/file/a9d751e6-35a8-4ef9-af90-f566b91a1834.pdf)

When we take a look at the public snippets of the manual (linked above), we can see the parameters table:

[![](images/09f8413138d9.png)](images/02e44cf07e42.png)

Walking through the first few parameters, we have our expiry date - obviously, this is a 2D surface, so we need to make one for each expiry.

Our ATM forward, which is the forward price, typically represents some sort of future that we will be settling to—this is, of course, different from the current spot price.

SSR is a parameter for how responsive we should be to adjustments in the price and our skew. It’s usually set at 0 by default, and it’s fine to leave it there.

Our reference volatility is the overall volatility (usually we fit to the low-point, i.e. the lowest implied volatility for that expiry), which we can adjust to shift our curve up and down, as shown below:

[![](images/1818d8fddb16.webp)](images/fa72dc1569f7.webp)

We can see as we increase the volatility reference (vr), the curve shifts up.

Next, we get to the VCR, or volatility change rate. By default this is set at 0, but it effectively is how we should change our volatility given changes in spot. Generally, practitioners will fit this using a tangent to the ATM:

[![](images/e7ee4c203653.png)](images/24caee4317b9.png)

We can the interpolate across this to shift our volatility up:

[![](images/45f5bd6a7dce.png)](images/06eaa22f146f.png)

As seen above, we shift our volatility along this tangent to reflect the change in spot price. This only works for small ranges, and typically we have a second derivative in more advanced models where any large move in either direction will slope our tangent exponentially up:

[![](images/16b59a41939f.png)](images/3c61af46a04e.png)

We shift along the exponential line, which is roughly linear for small moves, but of course, if we go -50%, we don’t want to decrease our volatility just because we had a spot/vol correlation that was pointing upwards.

[![](images/e73b83319206.png)](images/ca92588b334c.png)

There are many more parameters; SCR is the same concept as VCR, but for the central skew point of the curve (i.e., how do we shift this with spot - or rather the ATM forward for the wing model). We also set this to 0 by default - same as VCR, but also like VCR we probably don’t want to leave it at 0 if we care about improving our fit.

There are also a few settings related to the cutoff point for up and down, the put and call curvature, lots of parameters that I won’t go into.

[![](images/a24837eca6a6.png)](images/e34db913d379.png)

Here are the volatility skew settings as a table; all of this can, of course, be found in the PDF, which was attached above.

Finally, I leave you with the code to play around with this:

[![](images/5727c919855f.png)](images/3d5d3f3e1180.png)

#### Conclusion

---

Hopefully, this has been an informative article that gives readers some practical code to go home with.

Feel free to reach out on Twitter with questions as always :)
