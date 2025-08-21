# Risk_Simulation_Using_Heston_Model

## Introduction.

Predicting security prices in the capital markets is not an easy task, and you can never be 100% accurate about future prices. However, this does not mean that you cannot forecast the risks associated with these securities. There are various methods to simulate such risks, one of which is the Heston Model combined with Monte Carlo Simulation. This project applies that approach and demonstrates how the Heston Model is more reliable than models that assume constant risk when it comes to estimating future prices.

## What is Heston Model?

The Heston Model is a stochastic pricing model that incorporates varying levels of risk to forecast the future prices of a security. Unlike models that assume constant risk, it is more realistic since market risks are never truly fixed. This makes the Heston Model a more reliable and accurate tool for simulation purposes.

## How it works.

The Heston Model incorporates varying levels of risks, we use the following formula to calculate the varyings risks.

here is the equation to calculate the change in risk:

$$
dv_t = \kappa \cdot ( \theta - v_t) \cdot dt + \xi \cdot \sqrt v_t \cdot dW^2_t
$$

here is the quation to calculate the change in price:

$$
dS_t = \mu \cdot S_t \cdot dt + \sqrt v_t \cdot S_t \cdot dW^1_t
$$

There exists a relation between $dW^1_t$ and $dW^2_t$ and the correlation, that is:

$$
dW^1_t \cdot dW^2_t = \rho \cdot dt
$$

### Terms used:

$\mu$ : Mean Returns.

$v_t$ : Instantaneous Variance.

$\theta$ : Long run mean Variance.

$\xi$ : Vol of Vol {Standard Deviation of Variances}.

$\rho$ : Correlation between prices and variances.

$\kappa$ : Mean reversion rate.

Past 5 years data of the prices is used to calculate the values for the above variables, the data is exracted from the Yahoo Finance website using the yfinance library.
Euler's Numerical Scheme is used to simulate prices step by step, this is done to ensure that the structure of the continuous time model is maintained.

## Monte Carlo Simulation.

Monte Carlo simulation estimates all the possible outcomes of a process by running it several times with different inputs. instead of giving one particular prediction it gives us a distribution of possible outcomes, which can then be used to derive the probability of a certian outcome or a range of different outcomes.

Simulating the prices using Monte Carlo helps us understand:

- variaty of different pathways that the prices can move in. Running the simulation 10000 times will cover almost all the possible paths that the prices can move in an year.

- Calculate the risk measures like VaR and CVar and probable loss.

## Output from the Simulations

#### Price Movement.

![Price Movement](/Graphs/Price%20Movement.png)

In the Above line graph you can clearly see the all the possible prices that the model predicted/ calculated. We can calculate the risk measures using the histogram.

- The mean price of all the 10,000 final prices is **39,010.49**

- The 5% Value at Risk [ VaR ] is rupees 2144.09 which implies a final portfolio value of rupees 27855.91

- The Conditional VaR is rupees 4254.09 implying that the final portfolio value is rupees 25745.91

- The probability of loss in this case is 9.98%

#### Variance Movement.

![Variance Movement](/Graphs/Variance%20Movement.png)

The above graph shows the movement of variance at all the given time period, the variances keep on changing at the rate $\kappa$ and reverts back to the mean reversion rate $\theta$.
The varying levels of variance makes the Heston Model very reliable to predict the pathways.

## How does the Simulation work?

The first step in this process is calculating the values for all the parameters that the Heston Model uses, you can find the calculations in the [Colab Notebook](/Portfolio_Heston_Model.ipynb).

After all the Parameters are calculated we calculate the varying levels of variances:

```py
from math import exp
S0 = 30000
v_t = theta

var_paths = np.zeros( (M , days + 1) )
var_paths[:, 0] = v_t

price_paths = np.zeros( (M , days + 1) )
price_paths[:,0] = S0

final_prices = pd.DataFrame()
Final_vol = pd.DataFrame()

epsilon = 1e-8
for t in range( 1 , days + 1):
    vtml = np.maximum(var_paths[:, t -1] , epsilon)
    sqrt_v = np.sqrt( np.maximum(vtml, epsilon) )
    var_paths[:, t ] = var_paths[:, t-1] + kappa * ( theta - vtml ) * dt + Xi * sqrt_v * Z2[:, t-1] * np.sqrt(dt)
    var_paths[:, t] = np.maximum(var_paths[:, t], 0)
```

After the variances are calculated all the different prices that the Heston Model predicts using the varying levels of variances are calculated as follows:

```py
for t in range( 1 , days + 1):
      sqrt_v = np.sqrt(np.maximum(var_paths[:, t -1], epsilon))
      price_paths[:, t] = price_paths[:, t-1 ] * np.exp( ( mu - 0.5 * var_paths[:, t-1 ]) * dt + Z1[:, t-1] * sqrt_v * np.sqrt(dt) )
final_prices = price_paths[:, -1 ]
```

The Dataframe final_prices stores all the possible final prices after a year, we then visualizes this data.

## Problems and their solutions.

During the process, I faced various problems like:

- Exploding Variances

- Negative values

- Prices always moving upwards

To solve them, I used techniques like truncation (clipping negative variances), proper initialization with realistic parameters, and careful calibration to past data. I also adjusted the simulation steps to make sure volatility behaved correctly. These fixes helped the model work smoothly and give realistic results.

## Conclusion

In this project, we used the Heston Model to study how our portfolio might behave in the future. Unlike basic models, Heston model allows volatility to change over time and move back toward an average level, which makes it closer to real markets. By running Monte Carlo simulations and carefully setting the parameters, we could handle issues like negative variances and get more realistic results. This makes the Heston Model a reliable choice for risk analysis and portfolio forecasting.
