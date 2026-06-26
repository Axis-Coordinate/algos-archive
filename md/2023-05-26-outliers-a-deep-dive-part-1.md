# Outliers - A Deep Dive (PART 1)

Source HTML: [`html/2023-05-26-outliers-a-deep-dive-part-1.html`](../html/2023-05-26-outliers-a-deep-dive-part-1.html)

# Outliers - A Deep Dive (PART 1)

| 항목 | 값 |
| --- | --- |
| 날짜 | 2023-05-26 |
| 접근 | 무료 |
| URL | https://www.algos.org/p/outliers-a-deep-dive-part-1 |
| 부제 | Dealing with extreme values as a quantitative researcher. |

---

[![](images/6d56ff88c217.png)](images/f8ebae904143.png)

#### Introduction

---

There’s a lot of talk about outlier detection and removal, so today we’ll cover everything related to the subject to try to give a clearer picture. Outliers are a common problem every quant faces, but they also present an opportunity to find alpha. When greed and panic take over, the market becomes far more predictable. Outlier events are often when we get to take advantage of this. We explore how to use outlier events and prevent them from damaging your model.

The first half of this article series (PART 1) looks at outliers as our enemy and how we can deal with them. The other half (PART 2) focuses on how to model extreme values so we aren’t discarding some of the most information-dense data points.

It is my view that extreme events are best researched on their own. We thus want to reduce their extreme nature or even remove them when analyzing the complete data. Still, these data points aren’t simply thrown away - they are modeled separately. They often behave in ways entirely opposed to the usual data, requiring special treatment.

Quant’s Substack is a reader-supported publication. To receive new posts and support my work, consider becoming a free or paid subscriber.

#### Index

---

##### Part 1:

1. Outlier Detection

   1. Plotting data
   2. Quantile Regression
   3. IQR
   4. Z-Score Quantiles
   5. Clustering Approaches
2. Transforming Data

   1. Winsorising
   2. Log Transforms
   3. Min/Max Scaling
   4. Median Absolute Deviation
   5. Non-Linear Scaling
3. Robust Modelling Approaches

   1. Ridge
   2. LASSO
   3. PLS
   4. MAD
   5. The L0, L1, & L2 norm.

##### Part 2 (In the next article) :

1. General Thoughts
2. Feature Engineering

   1. Min/Max
   2. Quantiles
   3. Volume
   4. Volatility
   5. Return Discreteness

#### Outlier Detection

---

We will want to know the extent to which outliers exist in our dataset; detection is the first step. A lot of this is intuition, as we don’t just need to detect outliers. This is financial data - you can expect to find outliers; the real question is to what degree do they exist. Do they behave differently? Does this alpha have a sweet spot between too extreme and not extreme enough to beat fees?

\_**Plotting Data** \_is one of the simplest ways to do this. sns.regplot() should show us how extreme our outliers are. We can tell simply from where most of the data points sit relative to the size of the axis. Look at the most extreme values; do they cluster? Are there just one / a couple of significant outliers or many (relative to dataset size)? Quickly eyeballing it with some plots is an excellent way to start building an idea of how these outliers will behave.

***Quantile Regressions*** can be used to see if the relationship between our feature and our outlier returns is consistent compared to the rest of the data. If the relationship is the same, we should focus on the outliers, not remove them. We want the most extreme values because they will create the highest EV - but of course, only in this scenario where the relationship is consistent.

***IQR*** is probably one of the best-known techniques. It is high school-level statistics. I figured it deserved a mention, but it isn’t something I’ve found much use from.

***Z-Score Quantiles*** are another excellent way to go about it. We first apply a z-score transform and then print out the percentiles. If your 99.9% has a very high z-score, we know there are some outliers. This lets us get a clearer picture of how extreme our values are and how many outliers exist. If there are quite a few extreme samples, they’re more likely to affect our model, but it also means we have a much better chance of finding alpha in the extremes. One sample is insufficient to make an alpha; having a few samples gives us some room to work.

***Clustering Approaches*** like ***DBSCAN*** are another way to find outliers. This is less useful for detection / exploratory inference, for which the rest of these techniques are suited. We’ll come back to this in the second part, as this is best used to find alphas, not necessarily to understand your data and its outliers.

#### Transforming Data

---

Now that we’ve covered some general ways to gain inference into how many outliers your data has and what form they take, we can move on to how to deal with them. These are transforms that can be used to reduce the extreme nature of our outliers.

***Winsorising*** is probably my go-to approach for this. We cap data points that have more than a certain absolute z-score or alternatively remove them entirely. I prefer to cap the values, but you can experiment with different forms.

***Log Transforms*** are the second most popular way to deal with non-normal distributions. This is more appropriate for some things than others, so it can be a bit hit or miss. Understanding the mathematics behind this transform is a good idea so you can apply it when it is most appropriate. You risk throwing away information, especially for making data stationary, fractional differencing can make sense instead.

***Min/Max*** normalization is the complete opposite of the previous two. This actually highlights outliers and gives them more weight. It’s a valuable normalization because all values fall within a 0-1 range. You subtract the minimum, then divide by maximum - minimum. Thus, your min has a value of 0, and your max has a value of 1. This is usually appropriate when there is a clear reason (such as forced hedging) where the magnitude should have a linear relationship with EV.

***Median Absolute Deviation*** normalization is a technique I really like; I find that it is best paired with Winsorising the data. We subtract the median and then divide it by the median absolute deviation.

***Non-Linear Scaling*** is more of an idea I’ve tested out, I don’t think it’s very established in the literature, but the idea is that you have a threshold where you aggressively start scaling down the values. For example, past 3SD, we use a log transform. This would have the formula: (3 + log(SD - 2)), but it is really something you tune manually. It is a less aggressive method than Winsorising and similarly preserves most of the data.

#### Robust Modelling Approaches

---

I’ll cover some popular models here, but there is much more to this subject than I’ll go into. It’s worth understanding how shrinkage works at a very intuitive level to make use of these regression techniques best.

***Ridge*** regressions are fantastic and are usually my default choice. Regularizing your features is very valuable, but I prefer Ridge over LASSO simply because if a feature is in my model, it should have some non-zero weight. If it needs to be excluded that is usually a piece of research that happens before it goes into the model.

***LASSO*** regressions are also quite good, and my view that no features should be excluded is not always something others will agree with, so I’m sure there are many viewpoints. Still, I find it more efficient to research each feature and not include weak features in the first place. There are certain scenarios where I find LASSO useful, however, and that is usually when I feel that a specific feature or group thereof that only apply to certain assets. So in this case where I train the regression on each asset individually then this may be the tool of choice.

***PLS (Partial Least Squares)*** is something I tend to go for when researching longer-term alphas, into the factor modelling literature. This or Ridge. It is excellent at dealing with multi-collinearity which is a big problem when you have lots of similar factors (many of which will be linear combinations of others if you just use them out the box - i.e. factor zoo).

***MAD (Mean Absolute Deviation)*** regressions are robust to outliers, but they aren’t always easy to fit and may have multiple solutions. To solve this I just increase the norm until it eventually fits properly, even if that is at L2.

The idea of norms is essential to a lot of this. I like building a custom optimization engine that lets me pick the norm and tune it to the level of outliers I see in the data. When you use an OLS regression, the error is squared then the average is square-rooted. This means that outliers actually get more weight - not less. MAD will weight them equally, but we can actually penalize for outliers by decreasing the norm. L0 is of course impossible, but we can use the square root of absolute error as a way to add a penalty for large values. Results tend to be far more robust out of sample when using the L1 norm. These ideas can be applied to PCA as well with RPCA (Robust Principal Component Analysis).

#### Conclusion

---

We’ve covered a lot in this article so I’ll leave part 2 for the next article, but hopefully, this gives some guidance on what to look at when working with outliers in your data.

Quant’s Substack is a reader-supported publication. To receive new posts and support my work, consider becoming a free or paid subscriber.
