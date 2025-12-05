# Multivariate-Energy-Load-Forecasting-VAR

## Page 1

PGR304 1

Dataset 1

Statistics 1

Load(MW) 1

Temperature 2

Price 2

Solar 2

Clouds 2

Distributions 2

Rolling statistics and the need for scaling 3

Load rolling mean and variance 4

Price rolling mean and variance 5

Solar rolling mean and variance 5

Cloud rolling mean and variance 6

ACF & PACF 7

Load 7

Temperature 8

Price 8

Solar 9

Cloud 9

Stationarity tests (ADF & KPSS) 10

Optimal differencing order 11

Model Selection 12

Granger Causality & CCF 14

Optimal VAR order 15

Summarization of VAR 15

Summarization of ARIMA 16

---


## Page 2

PGR304

This report analyzes a multivariate time series dataset containing records of observations between the time periods 2015-2018. The contents can be described as energy-related records. The goal of this report is to find the most optimal parametric model to forecast the last 48 hours of recorded “Load”. The series displays strong seasonal patterns, requiring extensive stationarity diagnostics using rolling statistics, ACF/PACF, ADF, and KPSS tests. After identifying suitable transformations, lag and variable analysis are performed using granger causality and cross-correlation. The report finally compares a univariate ARIMA with a multivariate VAR model.

Dataset

The dataset contains recorded variables load (load_MW), temperature (temp_C), cloud cover (cloud_pct), and price. The target variable is hourly energy demand (LoadMW), and the primary objective is to identify which independent variables best predict it. Load is measured in megawatts (MW), temperature in degrees Celsius, cloud cover as a percentage (0-100), solar generation in megawatts (MW), and price in euros per megawatt hour (€/MWh). Providing an overview of these units is essential, as their scales directly influence data interpretation.

Initial inspections of the dataset showed that 54 variables were missing, with 36 in the load column and 18 in the solar column. Because this is an hourly time series, introducing gaps by dropping these values would break the temporal structure. To maintain structure and preserve the hourly frequency, the timestamp column had to be converted to a correct datetime index, and the missing values had to be imputed using time-based interpolation. Interpolation is a technique in time series analysis that estimates missing data points by leveraging surrounding data, thereby imputing values that preserve the natural evolution of the time series (Hyndman and Athanasopoulos 2018). Missing data at such low percentages has little to no effect on the finale predictions, making imputation a valid option. After data imputation, the data set was finalized at 35,064 hourly observations. With that, the dataset provides a complete sequence of hourly observations from January 2015 through December 2018. This aligns with the expected number

---

Hyndman, Rob J., and George Athanasopoulos. 2018. Forecasting: Principles and Practice. OTexts. https://otexts.com/fpp3/.

---


## Page 3

of hours during that period, accounting for the leap day, making it a suitable dataset for time-series analysis.

## Statistics

### Load(MW)

The target variable, Load, has a mean of 28,700 MW and a standard deviation of 4,576 MW. This substantial spread, approximately 16%, indicates potential influence from daily or seasonal fluctuations. The minimum recorded value is 18,041 MW, and the maximum is 41,015 MW. The large variance reflects seasonal fluctuations and variations in demand.

### Temperature

The temperature variable has an average of 17.63°C and a standard deviation of 7.23°C. The minimum recorded temperature is -4.31°C, and the maximum is 38°C, indicating that the dataset originates from a region with a generally warm climate. Since temperature directly affects heating and cooling, the variable is expected to be correlated with load at certain lags.

### Price

The price variable has an average of 49.9 €/MWh and a standard deviation of 14.6, with values ranging from 2.06 to 101.99 €/MWh. Such variability is typical for electricity prices, which are subject to significant fluctuations. Price may contain substantial information about load shifts and direction.

### Solar

Solar generation averages 1.433 MW with a standard deviation of 1.679 MW, ranging from 0 to 5,792 MW. As solar energy is highly influenced by time of day and season, its contribution may be limited once seasonality is considered.

### Clouds

The cloud cover variable has a mean of 20.7%, a standard deviation of 25.6%, and spans the full range from 0% to 100%. Telling us that this variable varies a lot and is hard to predict.

---


## Page 4

The summary statistics indicate that the dataset contains substantial variations across variables, and this most likely reflects the daily and seasonal patterns common in electricity demand and weather-related observations. Load spans a wide range depending on demand, which is directly linked to heating and cooling periods. Temperature cements showing variability within cold and warm ranges. Solar generation and cloud cover both vary across large portions of their possible ranges. Price displays considerable volatility, unsurprisingly. All these observations prompt a closer inspection of the underlying distributions.

## Distributions

&lt;img&gt;KDE plots for load, temp, price, solar, clouds&lt;/img&gt;

Kernel density estimation (KDE) plots were utilized to further examine the variables. KDE is a non-parametric smoothing technique that reveals the underlying distribution of a variable without assuming a specific distributional form, such as normality (Silverman 1986). This method is particularly valuable in time series analysis, as it provides a visual summary of statistical properties. Additionally, skewness and kurtosis were calculated for each variable to enhance understanding of their distributions.

The **load** variable is nearly symmetrical, exhibiting low skewness (0.062) and mild kurtosis. Its KDE reveals a bimodal structure, likely resulting from seasonal demand fluctuations.
Temperature is also close to symmetrical, with a skewness of 0.038 and mild kurtosis (2.323), its KDE displays a bell-shaped distribution. **Price** demonstrates a mild left skew and a pronounced lower tail, consistent with electricity market dynamics where oversupply can lead to low prices. **Solar** generation is strongly right skewed, with pronounced daytime peaks, while cloud cover exhibits strong right skew and heavy tails. Cloud displays a multimodal and irregular distribution with several distinct peaks around 0%, 30%, 50 and 70%. These peaks reflect common weather states ranging from light to heavily cloudy.

---


## Page 5

The KDE analysis shows that the variables differ greatly in their distributional shapes. Some are nearly symmetric, while others exhibit strong skewness or multimodality. The observations indicate that the dataset likely is heterogenous and non-stationary, however, only covers the marginal distribution and not how each variable evolves over time. To evaluate trends and seasonal cycles which are critical to time series forecasting, the analysis has to cover rolling mean and variance properties.

# Rolling statistics and the need for scaling

&lt;img&gt;Rolling Mean & Variance of load_MW&lt;/img&gt;

The parametric, linear time-series models (VAR and ARIMA) require underlying data to satisfy key assumptions, the most important of which is stationarity: Stationarity implies a time series whose statistical properties dont depend on the time of measurement (Hamilton 1994). It is crucial to ensure a constant mean, variance, and stable autocorrelation structure over time. Failure to comply with these assumptions may lead to spurious correlations in the model's interpretations.

Rolling statistics for mean and variance were calculated to assess trends and changes in variability for each variable. Due to differing scales—load in tens of thousands of megawatts, solar in thousands, temperature in tens, and clouds on a 0-100 scale—the rolling plots were dominated by variables with the largest magnitudes. This scale of disparity also affected direct comparisons of mean and variance across variables.

To address this issue, min-max scaling was applied to each variable prior to plotting rolling statistics. This normalization preserves the structure of each series while enabling direct comparison and interpretation on a common scale (Pedregosa et al. 2011). After scaling, seasonal patterns and shifts in mean values became more apparent.

---


## Page 6

# Load rolling mean and variance.

&lt;img&gt;Rolling Mean load_MW on Scaled Data&lt;/img&gt;

&lt;img&gt;Rolling Variance of load_MW on Scaled Data&lt;/img&gt;

The rolling mean of the load variable still showed a similar structure every year, although it wasn't as consistent. After scaling, the rolling mean of the load variable exhibited a recurring annual structure, though with some inconsistency, indicating a non-stationary mean. Seasonal shifts were prominent and repetitive. The variance fluctuated but remained relatively stable over the years. These observations suggest that the series likely violates the stationarity assumption in its mean. Consistently through the summer and fall, through the later months every year. This repeating pattern indicated seasonality in the series. Likewise, the rolling variance of the series followed a seasonal pattern, with higher variance during transitional periods and lower during stable conditions.

# Temperature rolling mean and variance.

&lt;img&gt;Rolling Mean temp_C on Scaled Data&lt;/img&gt;

---


## Page 7

Rolling Variance of temp_C on Scaled Data

&lt;img&gt;A grid of four subplots showing rolling variance over time (2015-2018). Each subplot has a black line labeled "Rolling Variance" with values ranging from 0 to 0.04.&lt;/img&gt;

The rolling mean of temperature shows a very consistent annual cycle, starting low by new years, peaking in the summer and finishing low in December. The patterns repeats with slight variations indicating string seasonality and that the mean is not constant. The variance remains relatively low but varies across the time period. These observations lead to the assumption that temperature most likely is non stationary.

Price rolling mean and variance.

Rolling Mean price_DA_EUR_MWh on Scaled Data

&lt;img&gt;A grid of four subplots showing rolling mean over time (2015-2018). Each subplot has a red line labeled "Rolling Mean" with values ranging from 0 to 0.8.&lt;/img&gt;

&lt;img&gt;A grid of four subplots showing rolling variance over time (2015-2018). Each subplot has a black line labeled "Rolling Variance" with values ranging from 0 to 0.04.&lt;/img&gt;

The price variable exhibited a more irregular pattern. The rolling mean lacked a consistent structure, displaying gradual drifts in certain years and a slight upward trend in 2016 and 2018. This behavior suggests non-stationarity, though it may not be driven by weather-related factors. The variance showed distinguishable spikes, likely corresponding to market shocks, while remaining relatively stable otherwise. Overall, these patterns indicate that the price series is non-stationary.

---


## Page 8

# Solar rolling mean and variance.

&lt;img&gt;A 2x2 grid of subplots showing rolling mean (red lines) and rolling variance (black lines) for solar generation data from 2015 to 2018. Each subplot represents a year (2015-2018). The top row shows "Rolling Mean solar_MW on Scaled Data", with the x-axis ranging from January 2015 to December 2016 for 2015, January 2016 to December 2017 for 2016, January 2017 to December 2018 for 2017, and January 2018 to December 2019 for 2018. The bottom row shows "Rolling Variance of solar_MW on Scaled Data".&lt;/img&gt;

Solar generation exhibits the most pronounced seasonal movement after temperature, with its rolling mean and variance closely mirroring the temperature pattern. The rolling mean increases during summer and decreases in winter, reflecting changes in day length. Similarly, variance peaks in summer and declines in winter. These observations indicate that the solar generation series is non-stationary.

## Cloud rolling mean and variance.

Cloud is spiky because cloud conditions can change rapidly over short periods. This volatility appears clearly in the rolling variance, which spikes whenever the atmosphere shifts between clear, partly cloudy, and overcast states. Unlike the other weather variables, cloud cover does not follow a clear and consistent seasonal pattern. These abrupt transitions, combined with year-to-year irregularity, making it the most unpredictable of the weather variables.

&lt;img&gt;A 2x2 grid of subplots showing rolling mean (red lines) for cloud cover data from 2015 to 2018. Each subplot represents a year (2015-2018). The top row shows "Rolling Mean clouds_pct on Scaled Data", with the x-axis ranging from January 2015 to December 2016 for 2015, January 2016 to December 2017 for 2016, January 2017 to December 2018 for 2017, and January 2018 to December 2019 for 2018.&lt;/img&gt;

---


## Page 9

Rolling Variance of clouds_pct on Scaled Data

&lt;img&gt;Graphs showing rolling variance over time for 2015, 2016, 2017, and 2018.&lt;/img&gt;

# ACF & PACF

Analysis of rolling mean and variance served as an initial step in assessing stationarity. Earlier results indicated that all variables likely possess a unit root, with means drifting seasonally and variances fluctuating throughout the year. Visual inspection alone is insufficient for conclusive judgement; further analysis using autocorrelation and partial autocorrelation functions (ACF, PACF) is required. ACF and PACF illuminate the memory structure of a series and help determine the extent and type of dependence present (Box, Jenkins, and Reinsel 2008). These functions also reveal the need for, and degree of differencing required.

## Load

The ACF for the load variable exhibited strong positive autocorrelation, starting near 1 and decaying slowly. A clear pattern emerged at every 24th lag, with this structure persisting up to 500 lags. The PACF showed a prominent spike at lag 1 and weaker, yet noticeable, oscillations at multiples of 24. These findings support the assumption that the load series likely contains a unit root.

&lt;img&gt;ACF and PACF of load_MW&lt;/img&gt;

---


## Page 10

Temperature

The ACF for the temperature series remained consistently high. The plot displayed a repetitive wave structure indicative of a daily cycle. The PACF exhibited a large spike at lag 1, followed by a sequence of oscillations that gradually disappear. There is a clear seasonal daily cycle in the ACF. The PACF has a dominant spike at lag1 implying a direct AR(1) relationship. The temperature also is seasonal.

&lt;img&gt;ACF and PACF of temp_C&lt;/img&gt;

Price

The ACF for price started near one and declines rapidly. However, this decay is not consistent or fast enough to indicate stationarity. A weak repeating pattern can be noticed by the multiples of 24 reflecting daily patterns. The ACF also displays weaker, noisier oscillations, potentially reflecting market patterns and daily demand cycles. The PACF indicated short-term dependence, and both ACF and PACF exhibited irregular spikes at multiples of 24. These observations suggest that the price series is likely non-stationary, though not solely due to seasonality even though it is present.

---


## Page 11

ACF and PACF of price_DA_EUR_MWh

&lt;img&gt;ACF up to lag 50&lt;/img&gt;
&lt;img&gt;ACF up to lag 100&lt;/img&gt;
&lt;img&gt;ACF up to lag 500&lt;/img&gt;

&lt;img&gt;PACF up to lag 50&lt;/img&gt;
&lt;img&gt;PACF up to lag 100&lt;/img&gt;
&lt;img&gt;PACF up to lag 500&lt;/img&gt;

# Solar

The autocorrelation structure of solar seemed to be entirely controlled by the daily structure. Across all lags in the ACF plots, the series shows a clear wave pattern with a repeating phase every 24 lags. The spikes can be seen at lags 24, 48, 72, etc. The correlation between these points drops dramatically, indicating that solar is highly predictable within the daily pattern. The variable holds seasonality and is therefore likely non-stationary.

ACF and PACF of solar_MW

&lt;img&gt;ACF up to lag 50&lt;/img&gt;
&lt;img&gt;ACF up to lag 100&lt;/img&gt;
&lt;img&gt;ACF up to lag 500&lt;/img&gt;

&lt;img&gt;PACF up to lag 50&lt;/img&gt;
&lt;img&gt;PACF up to lag 100&lt;/img&gt;
&lt;img&gt;PACF up to lag 500&lt;/img&gt;

# Cloud

The cloud cover series exhibited distinct behavior compared to the other variables. Its ACF and PACF indicated erratic patterns with no discernible structure. The ACF decayed rapidly, suggesting minimal short-term dependence on past values, and no cyclical patterns were

---


## Page 12

observed. Among the five variables, cloud cover demonstrated the weakest long-term dependence and the fastest autocorrelation decay, making it the closest to stationarity.

ACF and PACF of clouds_pct

&lt;img&gt;ACF up to lag 50&lt;/img&gt;
&lt;img&gt;ACF up to lag 100&lt;/img&gt;
&lt;img&gt;ACF up to lag 500&lt;/img&gt;

&lt;img&gt;PACF up to lag 50&lt;/img&gt;
&lt;img&gt;PACF up to lag 100&lt;/img&gt;
&lt;img&gt;PACF up to lag 500&lt;/img&gt;

## Stationarity tests (ADF & KPSS)

Visual exploratory analysis revealed clear trends, seasonal cycles, and patterns in most variables. Tools such as rolling mean/variance, ACF/PACF, and KDE suggested the presence of unit roots in several variables. However, visual inspection alone is insufficient for absolute conclusions. To formally assess stationarity and determine the need for further transformation, two complementary tests were employed: the Augmented Dickey-Fuller (ADF) and Kwiatkowski–Phillips–Schmidt–Shin (KPSS) tests. The ADF test evaluates the presence of a unit root, with rejection of the null hypothesis indicating stationarity.(Dickey and Fuller 1979). In contrast, the KPSS test assesses whether a series is stationary around a mean or trend, with rejection implying non-stationarity.(Kwiatkowski et al. 1992). Using both tests together provides a robust statistical foundation, as agreement strengthens the conclusion. Disagreement between the tests can guide whether differencing or detrending is required.(Schwert 1989).

The ADF and KPSS finalize the assumptions across all variables. Four out of five are found to be non-stationary. The only stationary variable found was cloud percentage.

ADF rejected non-stationarity across all variables, and the large test statistics and low p-values on their own back up this claim. This is where it's crucial to have another assessor, in this case,

---


## Page 13

KPSS. KPSS agreed with ADF strictly on cloud and claimed non-stationarity on the rest of the variables across all significance levels

<table>
  <thead>
    <tr>
      <th>Variable</th>
      <th>ADF</th>
      <th>KPSS</th>
      <th>Interpretation</th>
      <th>Final</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>load_MW</strong></td>
      <td>Stationary (reject H0)</td>
      <td>Non-stationary (reject H0)</td>
      <td>Conflicting tests → typical I(1) pattern</td>
      <td>Difference stationary → Difference it</td>
    </tr>
    <tr>
      <td><strong>temp_C</strong></td>
      <td>Stationary</td>
      <td>Non-stationary</td>
      <td>Conflicting tests → classic borderline trend/cycle</td>
      <td>Difference stationary → Difference it</td>
    </tr>
    <tr>
      <td><strong>price</strong></td>
      <td>Stationary</td>
      <td>Non-stationary</td>
      <td>Same pattern as above</td>
      <td>Difference stationary → Difference it</td>
    </tr>
    <tr>
      <td><strong>solar_MW</strong></td>
      <td>Stationary</td>
      <td>Non-stationary</td>
      <td>Again ADF stationary, KPSS non-stationary</td>
      <td>Difference stationary → Difference it</td>
    </tr>
    <tr>
      <td><strong>clouds_pct</strong></td>
      <td>Stationary</td>
      <td>Stationary</td>
      <td>Both agree</td>
      <td>Stationary</td>
    </tr>
  </tbody>
</table>

Combined results from the ADF and KPSS tests indicate that load, temperature, price, and solar variables are difference stationary and are predominantly influenced by seasonal structures.

## Optimal differencing order

To identify the appropriate differencing order, tests were conducted on four seasonal lags corresponding to the natural cycles observed in the data.

*   24 hours - daily cycle
*   168 hours - week cycle
*   672 hours - 4-week cycle
*   8760 hours - year cycle

For each order, KPSS and ADF were used to reassess the Stationarity of variables and determine whether a unit root was still present. It was discovered that the 24-order differencing, the smallest, transformed the variables into stationary ones. It was of interest to avoid higher orders

---


## Page 14

or uneven orders to avoid instability and over differencing of variables.(Box, Jenkins, and Reinsel 2008).

<table>
  <thead>
    <tr>
      <th>Variable</th>
      <th>ADF Result</th>
      <th>KPSS</th>
      <th>Joint</th>
      <th>Final</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>load_MW</td>
      <td>Stationary</td>
      <td>Stationary</td>
      <td>Both tests agree</td>
      <td>Stationary</td>
    </tr>
    <tr>
      <td>temp_C</td>
      <td>Stationary</td>
      <td>Stationary</td>
      <td>Both tests agree</td>
      <td>Stationary</td>
    </tr>
    <tr>
      <td>pric</td>
      <td>Stationary</td>
      <td>Stationary</td>
      <td>Both tests agree</td>
      <td>Stationary</td>
    </tr>
    <tr>
      <td>solar_MW</td>
      <td>Stationary</td>
      <td>Stationary</td>
      <td>Both tests agree</td>
      <td>Stationary</td>
    </tr>
    <tr>
      <td>clouds_pct</td>
      <td>Stationary</td>
      <td>Stationary</td>
      <td>Both tests agree</td>
      <td>Stationary</td>
    </tr>
  </tbody>
</table>

## Model Selection

Once the relevant variables were rendered stationary, the focus shifted to selecting the optimal model for forecasting the final 48 hours of the Load series. The available modeling options included:

*   Univariate analysis: The Arima model that accesses past load values
*   Multivariate analysis: VAR model that takes independent variables into account

Prior to differencing, a correlation matrix of the variables' relationships was created.

---


## Page 15

Correlation Matrix

&lt;img&gt;
A correlation matrix heatmap with variables on both axes (load_MW, temp_C, price_DA_EUR_MWh, solar_MW, clouds_pct) and values ranging from -0.4 to 0.4. The color scale on the right indicates the correlation strength, with red representing positive correlations and blue representing negative correlations.
&lt;/img&gt;

The correlation matrix revealed several moderate relationships among the variables. Load exhibited the strongest linear association with price (0.47), followed by solar (0.4) and temperature (0.22). Cloud cover showed a near-zero correlation with load at 0.22. While these variables may initially appear to influence short-term load behavior, such correlations are largely driven by shared seasonality. Since all variables follow predictable daily and weekly cycles, contemporaneous correlation cannot distinguish genuine predictive relationships from those arising due to synchronized seasonality.(Enders 2014). To address this limitation, two additional tools were employed to capture lag-dependent relationships.

<table>
<thead>
<tr>
<th>Contemporaneous only</th>
<th>Correlation only measures same-time movement.</th>
<th>Cannot detect whether temp/solar predicts load.</th>
</tr>
</thead>
<tbody>
<tr>
<td>Inflated by seasonality</td>
<td>Shared daily/weekly patterns create artificial correlations.</td>
<td>Solar & temp appear predictive but lose significance after differencing.</td>
</tr>
</tbody>
</table>

---


## Page 16

<table>
  <tr>
    <td><strong>Symmetric measure</strong></td>
    <td>Correlation cannot distinguish X→Y from Y→X.</td>
    <td>Cannot tell whether price drives load or vice versa.</td>
  </tr>
  <tr>
    <td><strong>Sensitive to non-stationarity</strong></td>
    <td>Trending series correlate even without real linkage.</td>
    <td>Weather variables look important before differencing, irrelevant after.</td>
  </tr>
  <tr>
    <td><strong>No lag structure</strong></td>
    <td>Forecasting requires knowing which lags matter.</td>
    <td>CCF shows only price retains predictive lagged signal.</td>
  </tr>
</table>

Since correlation alone cannot determine which variables most effectively predict load, further assessment was conducted using the cross-correlation function (CCF) and the Granger causality test. It is important to note that both CCF and the Granger causality test assume stationarity. (Granger 1969; Lütkepohl 2005). Failure to properly transform the data may result in misleading or spurious outcomes.

## Granger Causality & CCF

X Granger-cause load_MW?

<table>
  <tr>
    <th>Variable</th>
    <th>First Significant Lag</th>
    <th>Significant Lags</th>
    <th>Overall Causality Verdict</th>
  </tr>
  <tr>
    <td>temp_C</td>
    <td>Lag 2</td>
    <td>2, 3, 4, 5</td>
    <td>Weak–Moderate causal effect at short lags</td>
  </tr>
  <tr>
    <td>price</td>
    <td>Lag 1</td>
    <td>All lags (1–24)</td>
    <td>Very strong and persistent causal effect</td>
  </tr>
  <tr>
    <td>solar_MW</td>
    <td>Lag 1</td>
    <td>All lags (1–24)</td>
    <td>Causal effect present</td>
  </tr>
  <tr>
    <td>clouds_pct</td>
    <td>None</td>
    <td>None</td>
    <td>No causal effect detected</td>
  </tr>
</table>

Cross-Correlation: Which variables move with load, and how?

<table>
  <tr>
    <th>Variable</th>
    <th>CCF Peak (Magnitude)</th>
    <th>Peak Lag</th>
    <th>Relationship Type</th>
    <th>Interpretation</th>
  </tr>
  <tr>
    <td>temp_C</td>
    <td>-0.039</td>
    <td>Lags 10–14</td>
    <td>Weak negative</td>
    <td>Lower temperature → slightly higher load, but very small effect</td>
  </tr>
</table>

---


## Page 17

<table>
  <tr>
    <td><strong>price</strong></td>
    <td><strong>+0.506</strong></td>
    <td>Lag<br>0</td>
    <td>Strong<br>positive</td>
    <td>High price and high load co-move<br>strongly</td>
  </tr>
  <tr>
    <td><strong>solar_MW</strong></td>
    <td><strong>-0.039</strong></td>
    <td>Lags<br>18–<br>20</td>
    <td>Weak<br>negative</td>
    <td>Solar reduces load slightly (logical but<br>effect tiny)</td>
  </tr>
  <tr>
    <td><strong>clouds_pct</strong></td>
    <td><strong>-0.018</strong></td>
    <td>Lags<br>17–<br>20</td>
    <td>Very weak<br>negative</td>
    <td>Cloudiness barely relates to load</td>
  </tr>
</table>

The results indicate that, after enforcing stationarity, price and solar variables provide clear and robust predictive information for future changes in load. Temperature offers only weak short-term contributions, while cloud cover demonstrates no predictive value. Consequently, temperature and cloud were excluded from further analysis. As load was found to depend on at least two variables, a multivariate analysis approach was selected as the primary forecasting method.

**Optimal VAR order**

To determine the optimal VAR order, a two-phase search strategy was implemented, guided by the seasonal characteristics of the data. The initial phase involved fitting models at seasonal lags of 24 (daily), 168 (weekly), and 720 (monthly). For each candidate, BIC and AIC values were recorded and compared. The daily lag (24) was too short, while the monthly lag (720) was too long. The weekly cycle (168) performed best among these, prompting a more detailed search in neighboring orders. A grid search within the 0-200 range identified the lowest BIC and AIC values at 15.18 and 15.005, respectively. Order 146 was selected because it closely matched the weekly pattern in the data, yielded the lowest BIC (the preferred criterion for large samples), and BIC values increased consistently beyond this point.(Schwarz 1978).

**Summarization of VAR**

After model predictions were made, the test and prediction data were transformed back to the original scale using their 48-lag counterparts. The VAR(146) model achieved an RMSE of 1313MW, an MAE of 1100MW, and an MAPE of 4.01%. The mape is the most interpretable metric because the target data have large magnitudes. The 4% error indicates that the model

---


## Page 18

effectively captured the short-term dynamics and patterns. To better understand how forecast accuracy evolves over the 48-hour period, an hour-by-hour comparison of the difference between the prediction and the actual data was made. The pattern revealed that the model was more accurate early on, in the first 6-12 hours. As the range grows, so does the distance between truth and prediction

## Summarization of ARIMA

To evaluate whether a much simpler univariate forecasting model would be able to keep up with the multivariate VAR. The raw load variable was fitted onto an ARIMA model. Because the series becomes stationary at the seasonal differencing order of 24, a grid search was set within the lower search space of orders p,d,q we found AIC order 9,0,8 as the best performing configuration.

Order 9,0,8 stands for 9 order auto-regressive, 0 differencing and 8 order moving average. The ARIMA model had to search at a deep level to try to close the gap between it and the VAR. The ARIMA grid search was computationally heavy because every candidate model requires maximum-likelihood estimation. The search space used in this study contained 500 ARIMA configurations.(Box and Jenkins 1970; Hyndman 2015).

*   p ∈ [0, 9]
*   d ∈ [0, 4]
*   q ∈ [0, 9]

On a standard laptop, this search would take atleast a day. To make the procedure tractable. All models were evaluated using parallel processing across 96 vCPUS on high compute instance. The found configuration provided valid forecasts, but its error metrics were substantially worse than those produced by the VAR.

---


## Page 19

# Conclusion

The ARIMA (9,0,8) model produced a valid forecast, but its error metrics were substantially worse than those of the VAR model. Its performance can be summarized as RMSE: 4049 MW, MAE: 3460 MW, and MAPE: 12.3%. The univariate analysis shows that a single-variable model cannot capture enough structure in the load series, even when heavily tuned, because it ignores important variables that help explain load. The VAR model consistently outperforms ARIMA by a large margin, both statistically and visually, due to the advantage of incorporating the variables identified earlier as important predictors. The final model choice is therefore the VAR configuration.

# References

Box, George E. P., and Gwilym M. Jenkins. 1970. Time Series Analysis: Forecasting and Control. Holden-Day.

Box, George E. P., Gwilym M. Jenkins, and Gregory C. Reinsel. 2008. Time Series Analysis: Forecasting and Control. 4th ed. Wiley.

Dickey, David A., and Wayne A. Fuller. 1979. “Distribution of the Estimators for Autoregressive Time Series With a Unit Root.” Journal of the American Statistical Association 74 (366): 427–431.

Enders, Walter. 2014. Applied Econometric Time Series. 4th ed. Wiley.

Granger, C. W. J. 1969. “Investigating Causal Relations by Econometric Models and Cross-Spectral Methods.” Econometrica 37 (3): 424–438.

---


## Page 20

Hamilton, James D. 1994. Time Series Analysis. Princeton University Press.

Hyndman, Rob J. 2015. “The State Space Approach to ARIMA Modeling.” arXiv:1508.01580.

Hyndman, Rob J., and George Athanasopoulos. 2018. Forecasting: Principles and Practice. 2nd ed. OTexts.

Kwiatkowski, Denis, Peter C. B. Phillips, Peter Schmidt, and Yongcheol Shin. 1992. “Testing the Null Hypothesis of Stationarity Against the Alternative of a Unit Root.” Journal of Econometrics 54 (1–3): 159–178.

Lütkepohl, Helmut. 2005. New Introduction to Multiple Time Series Analysis. Springer.

Pedregosa, Fabian, et al. 2011. “Scikit-Learn: Machine Learning in Python.” Journal of Machine Learning Research 12: 2825–2830.

Rosenblatt, Murray. 1956. “Remarks on Some Nonparametric Estimates of a Density Function.” Annals of Mathematical Statistics 27 (3): 832–837.

Schwarz, Gideon. 1978. “Estimating the Dimension of a Model.” Annals of Statistics 6 (2): 461–464.

Schwert, G. William. 1989. “Tests for Unit Roots: A Monte Carlo Investigation.” Journal of Business & Economic Statistics 7 (2): 147–159.

Silverman, B. W. 1986. Density Estimation for Statistics and Data Analysis. Chapman and Hall.

Wilks, Daniel S. 2011. Statistical Methods in the Atmospheric Sciences. Academic Press.
