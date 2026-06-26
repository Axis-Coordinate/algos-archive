# Seasonality in Commodities Markets

Source HTML: [`html/2023-06-09-seasonality-in-commodities-markets.html`](../html/2023-06-09-seasonality-in-commodities-markets.html)

# Seasonality in Commodities Markets

| 항목 | 값 |
| --- | --- |
| 날짜 | 2023-06-09 |
| 접근 | 무료 |
| URL | https://www.algos.org/p/seasonality-in-commodities-markets |
| 부제 | A full paper on seasonality co-authored with HangukQuant (Code Included) |

---

#### Introduction

---

Seasonality is a topic that I’ve found very interesting throughout my research, so I’m very excited to share with everyone a paper and I have co-authored together.

We explore classical seasonality and compare it to a novel generalized seasonality approach applied to commodities markets. This is a strategy that is boosted by improved execution but for the avoidance of assumptions we have refrained from simulating such a system. The beauty of a generalized framework is the fact that we can apply it almost anywhere, on any symbol, and in any asset class. A truly systematic approach.

The paper and code are included for all to access in the below sections. If you enjoyed this, feel free to check out both mine and 's other writings.

Quant’s Substack is a reader-supported publication. To receive new posts and support my work, consider becoming a free or paid subscriber.

#### Paper

---

Seasonality Paper

603KB ∙ PDF file

[Download](https://www.algos.org/api/v1/file/c5c378f0-69c6-48b2-9ca4-e79b1184eff8.pdf)

[Download](https://www.algos.org/api/v1/file/c5c378f0-69c6-48b2-9ca4-e79b1184eff8.pdf)

#### Code

---

```
def get_primes(n):
    #https://stackoverflow.com/questions/2068372/fastest-way-to-list-all-primes-below-n
    """ Returns  a list of primes < n """
    sieve = [True] * n
    for i in range(3,int(n**0.5)+1,2):
        if sieve[i]:
            sieve[i*i::2*i]=[False]*((n-i*i-1)//(2*i)+1)
    return [2] + [i for i in range(3,n,2) if sieve[i]]

def _get_feature_matrix(m,n):
    primes=get_primes(m+1)
    k=len(primes)
    dummies=[]
    for xk in range(k):
        arr=np.zeros(n)
        arr[primes[xk]-1::primes[xk]] = 1
        dummies.append(arr)
    X = np.vstack(dummies).T
    return X,primes,k


import re
import pytz
import numpy as np
import pandas as pd
import numpy_ext as npe
import matplotlib.pyplot as plt
import scipy.stats as scist
import statsmodels.formula.api as smf

from pprint import pprint
from datetime import datetime
from datetime import timedelta

contract, contract_df ="CL", pd.DataFrame() #retrieve data 
mgrps=contract_df.groupby(by=[contract_df.index.month, contract_df.index.year])
monthlies=pd.concat([(1+mgrps.get_group(group).cont_ret).cumprod().tail(1)-1 for group in mgrps.groups])
monthlies=monthlies.loc[sorted(monthlies.index)]

'''global in-sample seasonality test and portfolio on monthly returns
'''
X,primes,k=_get_feature_matrix(m=5,n=len(monthlies))
kwargs={"X_"+str(i):arr for i,arr in zip(range(k),X.T)} 
df=pd.DataFrame(index=monthlies.index, data={"r":monthlies.values, **kwargs}) 
multiple_tests={}
'''sequential regression testing'''
for j in range(k):
    formula=f"r ~ {' + '.join(['C(' + str(kwarg) +  ')' for kwarg in kwargs.keys() if int(kwarg.split('_')[1]) <= j])}"
    res=smf.ols(formula=formula,data=df).fit()
    j_test={
        "f_test": res.f_pvalue,
        "coeffs": res.params,
        "t_tests": res.pvalues
    }
    multiple_tests[j] = j_test

fpvals=[multiple_tests[j]["f_test"] for j in range(k)]
adj_pvals=k/np.sum([1/p for p in fpvals])
coeff_ps=multiple_tests[k-1]["t_tests"].values[1:]
coeff_vals=multiple_tests[k-1]["coeffs"].values[1:]
alpha=1 #ignore to show all in sample graphs

'''in-sample portfolio construction using the regression model'''
seasons=np.multiply(np.where(coeff_ps<alpha,1,0),coeff_vals)
votess = np.array([np.multiply(x,seasons) for x in X])
positions=np.array([np.sum(votes) for votes in votess])
print(contract)
print(round(adj_pvals,3))
print([round(p,3) for p in coeff_ps])
print([round(beta,3) for beta in coeff_vals], "\n")
df["positions"]=positions
df["insamp_ret"] = df.r*df.positions
plt.plot(np.log((1+df.insamp_ret).cumprod()),label="insamp_monthlies")
plt.title(contract)
plt.show()
plt.close()

'''walkforward classical seasonality test
based on last 5 years of data, long top 3 months and short bottom 3 months for the next one year period
'''
mgrps=contract_df.groupby(by=[contract_df.index.month, contract_df.index.year])
classical_df=pd.concat([(1+mgrps.get_group(group).cont_ret).cumprod().tail(1)-1 for group in mgrps.groups])
classical_df=classical_df.loc[sorted(classical_df.index)]
classical_df=pd.DataFrame(index=classical_df.index,data={"date": classical_df.index, "cont_ret": classical_df.values})
classical_df["positions"]=np.nan
def classical_roll(dt,r):
    df=pd.DataFrame(index=dt,data={"r":r})
    if not np.isnan(classical_df.at[dt[12*5],"positions"]):
        return
    calibrate_df=df.loc[:dt[12*5]] #first 5 year positions
    mnth_grps=calibrate_df.groupby(by=calibrate_df.index.month)
    mnthlies={grp: np.mean(mnth_grps.get_group(grp).r.values) for grp in mnth_grps.groups}
    mnthlies={k:v for k,v in sorted(mnthlies.items(), key=lambda pair: pair[1]) }
    short,long=list(mnthlies.keys())[:3],list(mnthlies.keys())[-3:]
    positions=[-1 if i.month in short else (1 if i.month in long else 0) for i in dt[12*5:]] #next one year positions
    classical_df["positions"].loc[dt[12*5:]] = positions

if len(classical_df)>12*6:
    rollop=npe.rolling_apply(
        classical_roll, 
        12*6,#5y calibration, 1 year walk
        *classical_df[["date","cont_ret"]].T.values
    )
    classical_df=classical_df.fillna(0.0)
    classical_df["classical_portfolio"]=classical_df.positions*classical_df.cont_ret
    ret_bnh=classical_df.cont_ret.loc[classical_df.cont_ret != 0].values
    ret_sea=classical_df.classical_portfolio.loc[classical_df.classical_portfolio != 0].values
    shrp_sea=round(np.mean(ret_sea)/np.std(ret_sea)*np.sqrt(12),3)
    shrp_bnh=round(np.mean(ret_bnh)/np.std(ret_bnh)*np.sqrt(12),3)

    plt.plot(np.log((1+classical_df.classical_portfolio).cumprod()),label=f"classical_seasonality: {shrp_sea}")
    plt.plot(np.log((1+classical_df.cont_ret).cumprod()),label=f"buy and hold: {shrp_bnh}")
    plt.title(contract)
    plt.legend()
    plt.show()
    plt.close()
```

Zipped Code (just change the file type back to .zip because Substack won’t attach zip files otherwise):

Script

1.91KB ∙ CBZ file

[Read now](https://www.algos.org/f/453eb286-b26e-4274-9d2d-75e5bc294f80.cbz)

[Read now](https://www.algos.org/f/453eb286-b26e-4274-9d2d-75e5bc294f80.cbz)
