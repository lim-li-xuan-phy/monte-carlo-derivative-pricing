# Overview
The **accuracy** of Monte Carlo simulations in computing the prices and Greeks of European and Asian options are **evaluated by comparison** against the Black-Scholes analytical values. As the random nature of the method adds uncertainty to the Monte Carlo estimate, an attempt was made to **improve the precision** of the estimate through antithetic variates. Results from Monte Carlo simulations were **close** to that from the Black-Scholes model with at most $\pm20$% error. **Uncertainty was lowered** by 25-30% with the help of antithetic variates. These findings suggest that Monte Carlo simulations can serve as an accurate alternative to calculating the prices of a diverse range of options which do not have a closed-form solution in the Black-Scholes model.

# Contents
- [Motivation](#motivation)
- [Methodology](#methodology)
- [Results and discussion](#results-and-discussion)
- [Limitations](#limitations)
- [Future work](#future-work)

*Derivations and financial concepts are explained in finer mathematical detail in `Analysis.ipynb`.*

# Motivation
The equation describing the price $C$ of a call option proposed by Fischer Black and Myron Scholes in 1973, 
$$C=S_t\Phi(d_1)-Ke^{-r(T-t)}\Phi(d_2)$$
$$d_1=\frac{\ln(\frac{S_t}{K})+(r+\frac{\sigma^2}{2})(T-t)}{\sigma\sqrt{T-t}}$$
$$d_2=d_1-\sigma\sqrt{T-t}$$
where $S_t$ denotes the underlying asset price at time $t$, $\Phi$ the cumulative probability density of a standard normal variable, $K$ the strike price, $r$ the risk-free interest rate of the underlying asset, and $T$ the time at which the option expires, is widely used by traders to compute the risk-neutral value of an option as a baseline to manage the amount of risk they are willing to take. The assumptions that this equation was founded on allow a well-defined solution to the price of European options, but are not suitable for other kinds of options such as options that have path-dependent prices or can be exercised early. Monte Carlo simulations enable the calculation of the prices of a  much larger class of options compared to the deterministic Black-Scholes model, although adding some uncertainty to calculations due to randomness. 

This project implemented a risk-neutral framework in the computation of option prices from the Black-Scholes model and Monte Carlo simulations. Interest rate $r$, volatility $\sigma$, and maturity duration $T$ were kept constant. Realistically, the parameters $r$ and $\sigma$ generally change as time $t$ progresses, affecting the underlying asset price $S_t$ and thereby leading to a different payoff than the value predicted in the risk-neutral case. Such a possibility of the option's actual returns being unequal to the forecasted returns is known as "risk". The level of risk associated with a financial derivative indicates the amount of potential financial loss that the buyer or seller could incur. Traders often use a set of measures called "Greeks" to quantify the risk of option contracts before their purchase. As a test of the accuracy of the Monte Carlo estimates, the Greek risk measures were computed and compared against the Black-Scholes analytical values.

# Methodology
## Monte Carlo simulations
Under the Black-Scholes model, the movement of asset prices is assumed to be governed by Geometric Brownian Motion, a random process. The theoretical basis of the Monte Carlo method is that by repeating simulations of possible paths many times, the empirical mean of the random simulated paths will approach the true value of the variable we want to measure as the number of simulations increase, according to the Law of Large Numbers. 

We utilise Monte Carlo simulations to compute the fair price of option premiums. A large number of probable price trajectories of the underlying asset are simulated, and the average payoff of the simulated paths will provide an estimate of the expected payoff of the option. After time-discounting, we can then obtain the fair price.

## Variance reduction
Monte Carlo standard error (MCSE) is a measure of the uncertainty of the variables estimated from Monte Carlo simulations. The MCSE for the mean of the target distribution is $\hat{s}/\sqrt{N}$, where $\hat{s}$ refers to the standard deviation and $N$ the number of paths simulated. A tenfold reduction in the MCSE needs one hundred times of the number of simulations, which would be computationally expensive at large $N$. Another way to lower the MCSE would be to reduce $\hat{s}$. There exist several variance reduction techniques to achieve that. We will be using one of them, antithetic variates, to decrease the variance of our Monte Carlo simulations.

## Option parameters
We concentrate on calculating the prices of call options because call and put option prices are interchangeable via the put-call parity. For our comparison of Monte Carlo results against the Black-Scholes exact values, the option parameters are kept at
$S_0=100, \mu=0.05, \sigma=0.2, T=1.0$, and $K = 100$. Every simulated path had 252 steps, to represent the number of trading days in a typical year.

# Results and discussion
The option premiums computed through the Monte Carlo approach were similar to the analytical values given by the Black-Scholes model. In line with the Law of Large Numbers, the percentage error drops as the number of simulations $N$ increase. These results suggest a high accuracy in the Monte Carlo estimates of option premiums at $N=10^3, 10^4,$ and $10^5$.

| Variable| Black-Scholes | N=10^3 | N=10^4| N=10^5 |
| --------| ------------- | ------ | ----  |------- |
| Premium | 10.45         | 10.6   | 10.59 | 10.48  |
| % Error | -             | 1.44   | 1.34  | 0.28   |

The Greeks values and variance reduction were assessed after setting $N=10^5$. While $\Delta$ and $\nu$ of the Black-Scholes model and Monte Carlo simulations yielded small percentage errors, the errors for $\Gamma$, $\Theta$, and $\rho$ were moderate. Running a greater number of simulations may be needed to improve the accuracy of the Greeks produced by the Monte Carlo simulations.

| Greek   | Black-Scholes value | Monte Carlo value | % Error    |
| ------- | ------------------- |------------------ |----------- |
| $\Delta$| 0.637               | 0.638             | 0.157      |
| $\Gamma$| 0.0188              | 0.0164            | 12.8       |
| $\Theta$| 6.41                | 6.96              | 8.58       |
| $\nu$   | 37.5                | 37.7              | 0.53       |
| $\rho$  | 53.2                | 63.8              | 19.9       |

Premiums of European call option (EUC) and average strike Asian option (ASC-S) were approximately equal before and after antithetic variates were introduced. This was consistent with the mathematical prediction that variance reduction does not affect the accuracy of the Monte Carlo simulations. MCSE fell as anticipated, showing that the use of antithetic variates has reduced the variance of our Monte Carlo estimates.

| Variable      | Without | With antithetic variates | % Fall    |
| ------------- | ------- |------------------------- |---------- |
| EUC Premium   | 10.48   | 10.47                    |-          |
| EUC MCSE      | 0.047   | 0.033                    | 29.8      |
| ASC-S Premium | 3.40    | 3.40                     |-          |
| ASC-S MCSE    | 0.016   | 0.012                    | 25.0      |

*All results are stored in `Results.csv`.*

# Limitations
- Monte Carlo method is less accurate than the exact analytical solution of the Black-Scholes model for pricing European options. However,  when a closed-form option pricing model is not available, Monte Carlo simulations would be able to provide useful estimates.
- Antithetic variates becomes less effective at reducing the variance of Monte Carlo simulations if the payoff function is non-monotonic because the negative correlation between the original payoff and its antithetic counterpart would decrease. Other strategies at lowering the variance, like stratified sampling and importance sampling, may be more suitable for such option pricing problems.

# Future work
- Monte Carlo method is flexible and can be implemented for calculating option prices of other kinds of options like American options or exotic options. 
- This project generated asset price trajectories from a pseudo-random number generator. For more realistic Monte Carlo estimates, the historical distribution of asset returns can be used instead to produce simulated price trajectories.