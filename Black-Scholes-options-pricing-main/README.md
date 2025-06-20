# Python Implementation and Analysis of the Black-Scholes Model for Pricing Call Options

## Synopsis


Tests the Black-Scholes model and its performance on forecasting option call prices of a selected option chain dataset. Discusses factors such as volatility and time to expiration that affect the estimations of call option prices and how this occurs within the dynamics of the model.

The model is implemented in [BlackScholes.py](https://github.com/dreamchef/black-scholes-options-pricing/tree/main/BlackScholes.py) The scripts in the [results_analysis](https://github.com/dreamchef/black-scholes-options-pricing/tree/main/results_analysis) folder generate the plots shown [in the following sections](https://github.com/dreamchef/black-scholes-options-pricing/tree/main?tab=readme-ov-file#analysis-of-results-predicted-vs-actual-call-prices). The [data](https://github.com/dreamchef/black-scholes-options-pricing/tree/main/data) folder contains the input data (scraped from Yahoo Finance using [option_scraperBS.py](https://github.com/dreamchef/black-scholes-options-pricing/blob/main/option_scraperBS.py)) and the output predicted data.

# Financial Background

Stock options are financial derivatives that allow one to buy or sell a stock at a predetermined price (strike price) on or before the specified expiration date; these instruments are employed by investors to hedge, speculate, and earn income. In particular, a call option is an option where you are able to buy a stock at the strike price after, in effect, betting that the stock will reach above this price during its expiry date. This leverage allows controlling more shares with less money - for the cost of the option. There are two types of call options: American-style calls and European-style calls. The difference between them is when the underlying security can be bought for the strike price in relation to the day it expires. American options can be exercised anytime before expiration. On the other hand, European calls can only be exercised at expiration. An Option Chain lists all available call contracts across stocks and expiration dates. Such a chain contains the data necessary for predicting prices using the Black-Scholes model.

# The Black-Scholes Model

The Black-Scholes model is a PDE-based financial mathematics model that provides a framework for option pricing. While it by definition prices European-style calls (as it does not bring into account the ability to exercise before the expiration date), it serves as an approximation to American-style call prices. The Black-Scholes equation is given by:

$$
\frac{\partial V}{\partial t} + \frac{1}{2} \sigma^2 S^2 \frac{\partial^2 V}{\partial S^2} + rS \frac{\partial V}{\partial S} - rV = 0
$$

where $V$ is the option price, $S$ is the stock price, $\sigma$ is the volatility, $r$ is the risk-free interest rate, and $t$ is time. In the following sections, this project explores the derivation of this equation and its solution in the case of European-style calls, its programmatic implementation to generate pricing predictions, and analyses of the results in the context of the model.

## Deriving the Model

Assumptions:

- The short-term interest rate is known and constant.
- The stock price follows a random walk in continuous time with a variance rate proportional to the square of the stock price. Thus, the distribution of possible stock prices at the end of any finite interval is log-normal. The variance rate of the return on the stock is constant.
- The stock pays no dividends or other distributions.
- There are no transaction costs in buying or selling the stock or the option.
- It is possible to borrow any fraction of the price of a security to buy it or to hold it at the short-term interest rate.
- There are no penalties to short selling. A seller who does not own a security will simply accept the price of the security from a buyer and will agree to settle with the buyer on some future date by paying him an amount equal to the price of the security on that date.

Letting $S_t$ be the price of a risky asset at time $t$ and $S_t^0$ be the price of a non-risky asset at time $t$, we can construct the stochastic differential equation that underlies the Black-Scholes model:

$$
dS_t = S_t(\mu dt + \sigma dB_t)
$$

$$
dS_t^0 = rS_t^0 dt
$$

where $B_t$ is a Brownian motion, $\mu$ is the mean rate of return, $r$ is the interest rate, and $\sigma > 0$ is the volatility rate. Brownian motion simulates the continuous time random walk that the stock price follows. Let $W_t = B_t + \frac{\mu-r}{\sigma}t$ and let the risk-neutral probability $P$ be defined with respect to $Q$, where $Q$ is the underlying probability function of the probability space where the Brownian motion lies.

$$
dP = exp\left(\frac{r - \mu}{\sigma}dB_s - \frac{1}{2}\left(\frac{r - \mu}{\sigma}\right)^2 ds\right)
$$

$W_t$ is still a Brownian motion, and $\frac{S_t}{S_t^0}$ is a martingale. $S_t$ must satisfy the following differential equation on $P$:

$$
dS_t = S_t(rdt + \sigma dW_t)
$$

Let us now consider a portfolio of $H_t$ risky assets and $H_t^0$ non-risky assets. The value of the complete portfolio at time $t$ is:

$$
P_t = H_t S_t + H_t^0 dS_t^0
$$

If we assume that the portfolio is self-financing or that any manipulation of $H_t$ or $H_t^0$ doesn’t require any inflow or outflow of money:

$$
dP_t = H_t dS_t + H_t^0 dS_t^0
$$

From this, we can tell that $\frac{P_t}{S_t^0}$ is also a martingale. Let $\phi$ be the payoff function, and a given time $T > 0$ be the maturity time. Our goal now is to build a portfolio $P_T = \phi(S_T)$. There are many options for $\phi$ depending on what we are dealing with. By the martingale representation theorem, we know that the answer is positive since $\frac{P_t}{S_t^0}$ is a martingale and $\phi(S_T)$ is $ F_t $-measurable. Thus, the portfolio has the following value at time $t$:

$$
P_T = E \left[ e^{-\int_t^T rds} \phi(S_T) | F_t \right]
$$

We should acknowledge that an analytical solution exists only for vanilla options and when $r$ and $\sigma$ are constant. If $r$ and $\sigma$ were functions of $t$ and $S$, we would only be able to evaluate $\phi$ numerically. Since $S_t$ is a stochastic process, it follows from the Markov Property that:

$$
P_t = p(t, S_t)
$$

where $p$ is a function of $t \in [0, T]$ and $S \in [0, \infty)$ and is the pricing function of the option. Additionally, we know that $p$ is a deterministic function. By the Markov Property of $S_t$:

$$
p(t, x) = E\left[ e^{-\int_t^T r ds} \phi(S_T) \bigg| S_t = x \right]
$$

where $(S_{\theta}^x)_t \leq \theta \leq T$ denotes the process solution to $dS_t = S_t(\mu dt + \sigma dB_t)$ starting from $x$ at time $t$:

$$
dS_{\theta}^x = S_{\theta}^x (r d\theta + \sigma dW_{\theta}), \quad \theta \geq t, \quad S_t^x = x
$$

Ito's lemma states that for a function $V(S, t)$ where $S$ follows a stochastic process defined by an SDE of the same form, the differential $dV$ satisfies:

$$
dV = \left( \frac{\partial V}{\partial t} + \mu S \frac{\partial V}{\partial S} + \frac{1}{2} \sigma^2 S^2 \frac{\partial^2 V}{\partial S^2} \right) dt + \sigma S \frac{\partial V}{\partial S} dW_t
$$

Now consider a hedging (risk-free) portfolio $\Pi$ consisting of one option and $-\Delta$ shares of stock where $\Delta = \frac{\partial p}{\partial S}$. The portfolio value is:

$$
\Pi = p - \Delta S
$$

The change in the portfolio value is:

$$
d\Pi = dp - \Delta dS
$$

Substituting $dp$ and $dS$ from above and assuming the portfolio to be risk-free (eliminating the $dW_t$ term):

$$
d\Pi = \left( \frac{\partial p}{\partial t} + \frac{1}{2} \sigma^2 S^2 \frac{\partial^2 p}{\partial S^2} \right) dt
$$

Since $\Pi$ is risk-free and its growth not perturbed by small changes in stock price, it should grow at the constant rate $r$, implying:

$$
d\Pi = r\Pi dt = r(p - \Delta S) dt
$$

Equating the expressions for $d\Pi$ and solving for $\frac{\partial p}{\partial t}$, we obtain the Black-Scholes PDE:

$$
\frac{\partial p}{\partial t} + \frac{1}{2} \sigma^2 S^2 \frac{\partial^2 p}{\partial S^2} + rS \frac{\partial p}{\partial S} - rp = 0, \quad p(T, S) = \phi(S)
$$

## Closed Solution of the PDE for European Calls

We could derive the closed form solution for a few other vanilla options but that is only a change in boundary conditions. So as an example of how it is done we will only show the example of a European call. The boundary conditions a European call imposes are:

$$
\begin{align*}
p(0, t) &= 0 \\
p(S, t) &\rightarrow S - \phi e^{r(T-t)} \text{ as } S \rightarrow \infty \\
p(S, T) &= \max(S - \phi, 0)
\end{align*}
$$

Of note is that our PDE is actually a Cauchy-Euler equation and thus we can change it into a diffusion equation with a change of variables. Specifically:

$$
\begin{align*}
\tau &= T - t \\
u &= Ce^{r\tau} \\
x &= \ln\left(\frac{S}{\phi}\right) + \left(r - \frac{\sigma^2}{2}\right)\tau
\end{align*}
$$

After this change of variables:

$$
\frac{\partial u}{\partial \tau} = \frac{\sigma^2}{2} \frac{\partial^2 u}{\partial x^2}
$$

The terminal condition that $p(S, T) = \max(S - \phi, 0)$ now becomes an initial condition.

$$
u(x, 0) = u_0(x) := \phi(e^{\max\{x,0\}} - 1) = \phi(e^x - 1)H(x)
$$

where $H(x)$ is the Heaviside step function. Now that we are seeing diffusion over an infinite domain and the Heaviside step function the natural conclusion of which method to use here is simple we use the convolution method:

$$
u(x, \tau) = \frac{1}{\sqrt{2 \pi \sigma^2 \tau}} \int_{-\infty}^{\infty} u_0(y) \exp\left(-\frac{(x - y)^2}{2\sigma^2\tau}\right) dy
$$

This can be algebraically manipulated into the form:

$$
u(x, \tau) = S N(d_1) - \phi e^{x+\frac{\sigma^2}{2}\tau} r N(d_2)
$$

where $N(\cdot)$ is the cumulative distribution function of the standard normal distribution and

$$
d_1 = \frac{1}{\sqrt{\sigma \tau}}
$$

$$
d_2 = \frac{1}{\sqrt{\sigma \tau}} \left(x + r - \frac{\sigma^2}{2}\right) + \frac{\sigma^2 \tau}{2}
$$

Lastly, we change variables back from $(u, x, \tau)$ to $(p, S, t)$:

$$
p(S, t) = S N(d_1) - \phi e^{-rt} N(d_2)
$$

# Obtaining Test Data for the Model

We proceeded to obtain data on which to test the model. While the closed-form solution is for European-style calls we can meaningfully test the model on American-style calls data since it is rarely optimal to buy before the exercise date. Such data is readily available as it is traded on large exchanges while European-style calls are often traded off-exchange and over-the-counter which would make obtaining such data exceedingly hard to find and aggregate. We collected input data that was generated on December 10th 2023 to be put into the model. We prepared a CSV file from the option chain data with columns for each of the required inputs to the Black-Scholes model: The CSV file was structured as in the following example:

| (Index) | Symbol    | Strike | (Bid) | (Ask)  | Volatility | Stock Price | Time to Exp. |
| ------- | --------- | ------ | ----- | ------ | ---------- | ----------- | ------------ |
| 0       | AAPL23... | 65     | 130.5 | 131.25 | 4.01...    | 195.71      | 4            |
| 1       | AAPL23... | 70     | 125.4 | 126    | 4.05...    | 195.71      | 4            |
| ...     |           |        |       |        |            |             |              |

It also contained the actual bid and ask prices which we use later to analyze the predictions. The risk-free interest rate as of December 10th 2023 was 4.23 percent [1]. We proceeded with the testing pipeline as shown in Figure 1. To use this data with the Black-Scholes formula we read the CSV file into the Python code (see appendix).  The code iterates through the time series and calculates call price for each call using the closed solution:

$$
C(S, t) = S N(d_1) - K e^{-rt} N(d_2)
$$

The code predicted prices for each option contract in the dataset. We then had to filter the data due to an issue with NaN values in the predicted prices. Upon initially finding this we expected it had resulted from either very low or high volatility very short time to maturity or extreme differences between the stock and strike prices. Upon examination of the scraped data set we noticed 0 to approximately 23.81 - a very wide range. After filtering out cases of extremely high volatility there were no remaining NaN price predictions and we were able to proceed to analysis.

<img src='figures/fig1.png' height='400' width='auto'>

# Analysis of Results: Predicted vs. Actual Call Prices

The following plots show the Black-Scholes predictions vs. actual prices:


<img src='figures/fig2.png' height='300' width='auto'>


<img src='figures/fig3.png' height='300' width='auto'>

## Correlation Between Actual and Predicted Ask/Bid Prices

The R-squared values calculated were as follows:

| Correlation Between           | Value  |
| ----------------------------- | ------ |
| Bid Price vs. Predicted Price | 0.7443 |
| Ask Price vs. Predicted Price | 0.7586 |

These suggest a high-moderate positive correlation between the predictions and actual call prices. However, the model is not nearly perfect. We proceeded to analyze how the model performs under varying market conditions, particularly differing volatility and time-to-expiration.

## Prediction Strength Under Varying Market Conditions

We separately grouped the set of predicted and actual prices into ranges over volatility and remaining time to expiration. For each subset, we calculated R² values.

## Volatility Analysis

We experimented with different groupings until we found the following in which there is an even distribution of option contracts between ranges. Figure 4 (on the following page) shows both the frequencies of options within range and the corresponding accuracies of predictions against bid and ask prices.
 Noticeably, the prediction strength is low for low volatility and high for high volatility. Recall that the Black-Scholes model assumes a log-normal distribution of underlying asset prices, constant volatility and risk-free rate, no dividends paid, and European exercise style. The predictions are influenced by how well these assumptions hold in each range.

In low volatility environments, the Black-Scholes model tends to be more accurate because the assumption of constant volatility is more closely aligned with market conditions. We would expect lower prediction errors in these ranges.

On the other hand, it is unrealistic to assume constant volatility in periods of high volatility. The Black-Scholes model assumes a log-normal distribution of stock prices and cannot accurately capture sudden large moves. We would, therefore, expect this to increase prediction errors.


<img src='figures/fig4.png' height='300' width='auto'>


## Remaining-Time Analysis

The data was then categorized into ranges determined in a similar fashion of values for time to expiration. Figure 5 shows the distribution of options across different time ranges and the corresponding R² values, showing how prediction accuracy varied over time. Again, how well the assumptions of the model hold in each time range influence the accuracy of the predictions.

<img src='figures/fig5.png' height='300' width='auto'>

The Black-Scholes model accounts for time decay through the time value component of the option price. Predictive accuracy might be higher as the model aligns well with the diminishing time value.

For options with longer times until expiration, the assumption of constant volatility is unrealistic. The volatility is likely to change, which leads to greater predictive errors.

# Experimental Error Analysis

## Near-Zero Predicted and Actual Prices

The most obvious anomalies include near-zero forecasts of significantly higher prices and relatively high forecasts of near-zero actual prices.
In total, there were 275 cases where such values for the model are zero while the actual quotes differ noticeably from zero. These options have a high average strike price of around 2149.38. Additionally, the average implied volatility is quite small (0.012), leading the model to underprice the options.
The model correctly predicted for high prices in 649 cases, whereas bids and asks came near zero. The average strike price is about 410.35 (more realistic), while the spread of strike prices is wider. These options show extremely low average implied volatility near zero, which may also mean they are deep out of the money or possess some other undesirable attributes on the market.

## Error from Volatility and Time to Expiration

Moreover, we examined what type of changing inputs could be related to greater error. Absolute difference between predictions and actual values, the bid and ask prices, and volatility and remaining time data served to compute the following correlation coefficients:

| Bid/Ask Error vs. Input          | Correlation Coefficient |
| -------------------------------- | ----------------------- |
| Bid Error vs. Implied Volatility | -0.151                  |
| Ask Error vs. Implied Volatility | -0.147                  |
| Bid Error vs. Remaining Time     | 0.039                   |
| Ask Error vs. Remaining Time     | 0.027                   |

The prediction errors are slightly negatively correlated with implied volatility and highly positively correlated with the remaining time until expiration.

The relationship of implied volatility and the remaining time until expira- tion and the accuracy of the predicted call prices is not linear. These observed correlations are small, implying that other determinants would probably pre- dict the accuracy of the Black-Scholes model more strongly.

# Summary and Conclusions

Our implementation of the Black-Scholes model frequently predicted near- zero price for significantly overpriced options. Such high strike prices coupled with very low implied volatility make it highly likely that the model under- priced the option. The actual evaluation of option value is affected many market factors and speculative trading strategies the model does not consider.
Most of the prediction errors can be tied back to assumptions that do not hold well in an actual market setting, such as constant volatility. Many market conditions and phenomena are not accounted for in the model.
Additionally, the since we used American-style option call data, a variety of factors related to calls being exercised before the expiration date are at play, without being accounted by the model and closed solution. At the same time, we did not ascertain to what extent such factors would affect the predictions accuracy.
These errors, in combination with our explanation of the assumptions inherent in the Black-Scholes model demonstrate the limitations of the Black- Scholes model.

## Future Research Directions

One direction that we could have looked into if we had more time would be considering exotic options. Many of them preclude a closed form solution, but they would have been a good direction to look at with the application side since we had already developed the theory. We also could have looked into models like Black-Scholes that don’t require constant volatility. The main downsides to considering most of these more complex models and options is that the theory is mostly the same, with the exception of no analytical solution, so this would have purely been a computational exercise.

# References

1. Ycharts 10-year treasury rate. [https://ycharts.com/indicators/10_year_treasury_rate](https://ycharts.com/indicators/10_year_treasury_rate). Accessed: 2023-12-10.
2. Yves Achdou, Olivier Bokanowski, and Tony Lelièvre. Partial differential equations in finance. The Encyclopedia of Financial Models, 2, 2012.
3. Fischer Black and Myron Scholes. The pricing of options and corporate liabilities. Journal of Political Economy, 81(3):637–654, 1973.
4. Martin Bohner and Yao Zheng. On analytical solutions of the Black–Scholes equation. Applied Mathematics Letters, 22(3):309–313, 2009.
