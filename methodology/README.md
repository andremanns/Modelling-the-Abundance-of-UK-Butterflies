# Project Methodology
## Contents

1. [Key Terms](#1-Key-Terms) <br>
	1. [Baseline Index](#i-Baseline-Index) <br>
	2. [Relative Abundance/Index Ratio](#ii-Relative-Abundance--Index-Ratio) <br>
	3. [Site/Species Combination](#iii-Site--Species-Combination) <br>
2.	[Model Selection](#2-Model-Selection) <br>
	1. [Model Structure](#i-Model-Structure) <br>
	2. [Model Distribution](#ii-Model-Distribution) <br>
	  	- [Profiling the Data](#Profiling-the-Data) <br>
	  	- [Selecting a Distribution](#Selecting-a-Distribution) <br>
3.	[Stage 1 GAM](#3-Stage-1-GAM) <br>
	1. [Data Requirements](#i-Data-Requirements) <br>
  	2. [Model Optimisation](#ii-Model-Optimisation) <br>
	3. [Code Links](#iii-Code-Links) <br>
4.	[Stage 2 GAM](#4-Stage-2-GAM) <br>
	1. [Data Requirements](#i-Data-Requirements) <br>
  	2. [Model Optimisation](#ii-Model-Optimisation) <br>
5.	[Bootstrapping](#5-Bootstrapping) <br>
6.	[Statistical Significance](#6-Statistical-Significance) <br>
7.	[License and Attribution Statement](#7-License-and-Attribution-Statement) <br>
8.	[References](#8-References) <br>

## 1. Key Terms
### i. Baseline Index
This is the count recorded in the baseline year (1993) for a single species at a single site.

### ii. Relative Abundance / Index Ratio
Relative Abundance (RA) and index ratio have the same meaning and are used interchangeably through the report. They describe the increase or decrease of a species count at a site in a specific year, relative to its baseline year. For example, if species code A received 8 counts at site code 1 in 1993, but 20 counts from the same site in 1997, the relative abundance for 1997 would be 2.5. Similarly, if in 2005, only 2 counts were recorded, the relative abundance for that year would be 0.25. This way a relative decrease is represented by values less than 1, and an increase by values greater than 1. The formula is shown below:

![Alt text](/images/methodology/ra_equation.png)

For small sample sizes RA is not very useful for summarising national trends. Natural stochasticity makes it difficult to identify the true signal, and location can also have a significant effect. However, when RA is combined over many different sites, it is possible to detect trends in some species. 

### iii. Site / Species Combination
Refers to a specific ‘survey grouping’ (a single species from a single site). For example, if 20 species are observed at site A, then 20 site/species combinations exist at this site. Similarly, if 20 of the same species are observed at sites A and B, then 40 site/species combinations exist across these two sites.

## 2. Model Selection
### i. Model Structure
GAMs were selected for their ability to capture non-linear trends - a characteristic associated with species abundance time-series data. Two explanatory variables were considered: time in years (assumed to be continuous for modelling); and species (categorical). The desired response variable was RA.

One issue to arise from this set up, was that many site/species combinations were not surveyed in the baseline year, limiting the sample size of RA training data for each species. This was a key consideration when deciding upon a suitable GAM structure. Several well-recognised options are evaluated in table 1.  


![Alt text](/images/methodology/gam_structure_table.png)
###### Table 1: Integers in GAM equations are used to represent the order of the input variables. Smooth terms are represented by s(), factor terms f() and random effects 're'.


Summarising table 1:
- Hierarchical GAMs, while preferred for their data sharing capabilities, were not feasible in Python.
- A fixed effect GAM would be ineffective for analysing species-specific trends.
- Both factor smooth models exhibited limited data sharing capabilities meaning a large proportion of survey data would be useless. Additionally, the sharing of hyperparameters in these last two models ignored the requirement for variable smooth complexity.

It was decided that two single-term-GAMs would be implemented. A ‘stage 1 GAM’ for predicting missing baseline counts, and a ‘stage 2 GAM’ for relative abundance trends. In both models the explanatory variable would be time (in years). Prediction of missing baseline counts would enable the calculation of RA for more site/species combinations, reducing the need for data sharing, a limitation of the factor smooth models. 

### ii. Model Distribution
#### Profiling the Data
Profiling UKBMS count data for all site/species combinations was not possible because absolute index scores from different groups were incomparable. Relative abundance profiling was possible, but baseline records were missing in some cases, and this method ignored count-level distribution. From other ecological abundance studies, it was noted that zero inflation, right skew, over-dispersion (a term used when variance exceeds the mean) and heteroskedasticity (variance increases with the mean) were characteristics typical of this data type [6], [7]. Another important, yet obvious observation was that count data cannot be negative. 
#### Selecting a Distribution
The negative binomial model appeared the most compatible. This model type is preferred in ecological count studies for its ability to fit datasets characterised by overdispersion and zero inflation [8], [9]. Unfortunately, this distribution was not available in Pygam. 

The Poisson model was available and stood out for its ability to predict integers that were non-negative [10]. However, Campbell et al. noted the potential for ‘serious errors’ in over-dispersed, zero-inflated datasets [11]. 

The Gaussian model seemed a poor fit considering its assumptions (normal distribution of residuals, and constant variance across a fitted range). In practice, when fitted to a heteroskedastic dataset, gaussian models fail to recognise noise at increased means, developing overly complex curves in these areas to minimise the sum of the squares. Squaring of the residuals, additionally has the effect of allowing large values to disproportionately influence the resulting smooth [12]. 

A solution was to log transform count data using log(x+ɛ) where ɛ, a small constant, was set to 1. This approach, similar to that used by Cao et al [13], enabled transformation of zero count data. The effects of log transformation were three-fold:
- Multiplicative differences between counts became additive. This ‘compressed’ the dataset (large values in particular) reducing right skew.
- The variance of counts reduced.
- Variance became more constant across the fitted range.

Better compliance with the Gaussian assumptions meant the data could be modelled after transformation. This method was approved for the stage 1 GAM (count data) and the stage 2 GAM (RA). 

To minimise bias and error associated with log transformation, species subsets and site/species combination groups were omitted if zero counts exceeded 15%.  This is approximately the threshold observed by B. O’Hara et al, below which the log(x+1) transformation exhibited the least error and bias when compared with other transformations in generalised linear models [14]. 

## 3. Stage 1 GAM
The aim was to predict abundance (the response variable) as a function of time (the explanatory variable in years) for site/species combinations without a baseline survey.

### i. Data Requirements
The data was first partitioned by site/species combination. Each grouping was required to meet the conditions outlined below:
- The number of ‘absent survey years’ between data points was limited to 5. This was to prevent GAM instability in regions of sparse data [15].
- At least 6 data points were required per group. The same minimum threshold was successfully used by ‘The Living Planet Index’ when predicting abundance trends [16].
- At least one record was required within five years of the baseline year.
- At least six records were required within ten years of the baseline year.
- To limit computational expense, records greater than 10 years from the baseline, were not used.
  
### ii. Model Optimisation
Optimisation was achieved by running models with different combinations of hyperparameters. The output baseline predictions were validated by comparing to the actual baseline values. A framework of the stage 1 GAM optimisation process is described below. Additionally, the corresponding code links are listed in table 2. 
1.	Site counts were grouped by species code and site code into ‘survey groups’. 
2.	A filter was applied to remove survey groups without a baseline record.
3.	Baseline indices were hidden from the remaining survey groups for later comparison.
4.	Counts were log-transformed (for details see section titled ‘[selecting a distribution](#Selecting-a-Distribution)’).
5.	A range of values were defined for each of the hyperparameters: lambda (eleven values log spaced between 10¯³ and 10³) and spline count (8 values between 4 and 11). This generated 88 hyperparameter combinations.
6.	A model was generated for each survey group using one of the hyperparameter combinations. The output of each model (a single baseline prediction per survey group) was stored in a DataFrame, containing all predictions for that set of hyperparameters.
7.	Step 5 was repeated for each combination of hyperparameters. 
8.	Residuals were computed by subtracting model predictions from the ‘hidden’ baseline values (see step 3).
9.	To determine model predictive power and bias, RMSE (Root Mean Square Error), correlation coefficient and ME (Mean Error) were computed for each of the hyperparameter combinations. Each of these metrics were visualised in terms of lambda and spline count. The aim here was not to select the best model, but to quickly identify the best candidates.
10.	The best performing models were compared using three visualisations:
	- Predicted vs actual baseline count
	- Fitted baseline count vs Residuals
	- Q-Q plot of residuals. 
11.	Following evaluation in step 10, the optimal hyperparameters were selected and implemented into the final stage 1 GAM.

### iii. Code Links
- [Running the model](/code/validation/valid_s1_gam/valid_s1_gam.ipynb) (relates to method steps 4-7).
- [Visualisation of hyperparameters](/code/validation/valid_s1_gam/12_13_14_obs/valid_s1_gam_12_13_14_obs.ipynb) (relates to method steps 8-9).
- [Evaluation of top performing models](/code/validation/valid_s1_gam/12_13_14_obs/valid_s1_gam_resid_12_13_14_obs.ipynb) (relates to method step 10).

## 4. Stage 2 GAM
The aim was to predict RA (the response variable) as a function of time (the explanatory variable in years) across all species subsets. Unlike the stage 1 GAM (which predicted counts for a single year) models were evaluated based on their ability to capture long term trends.

### i. Data Requirements
The dataset was divided by species with the minimum sample size set to 500. Species subsets exhibiting more than 15% zero counts, were removed (see model distribution, selecting a model for details). 

### ii. Model Optimisation
Models were run with different combinations of hyperparameters. Optimisation of the hyperparameters was achieved using three selection criteria: GCV (General Cross Validation), AIC (Akaike Information Criterion) and BIC (Bayesian Information Criterion). All three criteria were considered advantageous based on their ability to reward goodness of fit (sum of the squares for GCV, log likelihood for AIC and BIC) and penalise complexity (a function of the effective degrees of freedom). The method is outlined below:

1.	The data was partitioned into species subsets. 
2.	Relative abundance values were log transformed (for details see section titled ‘model distribution’).
3.	Subsets were randomly split into five ‘train/test’ sets. Note that data was randomly split, not split sequentially (as is common for time-series data). This is because smooths were intended for summarisation of existing historical data, not for extrapolation.
4.	The hyperparameters were defined: 5-6 basis functions, and 30 lambda values uniformly log spaced between 10¯³ and 10³. In total this produced 60 different hyperparameter combinations. 
5.	A model was generated for each hyperparameter combination, using the species training data. For GCV and AIC models, this was achieved through ‘gridsearch’, which automatically selected the optimal hyperparameters based on the input selection criteria. For BIC, an optimal model was identified using manually computed scores (BIC is not automated with gridsearch). 
6.	Optimised models (resulting from each of the model selection criteria) were used to generate predictions. Species test data was imputed into each of the models. 
7.	Model fits were visually assessed using the following methods:
	- Individual evaluation: models were assessed by overlaying test data, with the predicted relative abundance values. In this way it was possible to determine the goodness of fit.
	- Comparative evaluation: predictions from all models (log form) were plotted on the same graph.
	- Comparative evaluation: predictions from all models (following back-transformation) were plotted on the same graph.
	- Fitted values vs Residuals: residuals from the two preferred models were plotted on the same graph to compare magnitude of error and bias. 
	- Q-Q plots of the residuals. 
8.	Numerical assessment involved comparison of complexity (effective degrees of freedom), MAE (Mean Absolute Error) and RMSE. 
9.	Following evaluation, the optimal model selection criterion was determined and approved for use in the final analysis.

### Code Links
- [GAM stage 2 optimisation](/code/validation/valid_s2_gam/valid_s2_gam.ipynb)

## 5. Bootstrapping
Bootstrapping was used to estimate population central tendency (median RA) and variability (confidence intervals) for species abundance trends. Species subsets were resampled with replacement before running the stage 2 GAM. The process of resampling and modelling was integrated into a ‘for loop’ which repeated 500 times, meaning 500 RA predictions were made for each study year. 

## 6. Statistical Significance
The aim was to determine if increases or decreases in RA had occurred by chance. Four one-sided tests were performed with the confidence interval set to 0.95. The tests are listed below:

- Long term RA increase
- Long term RA decrease
- Short term RA increase
- Short term RA decrease

The method involved analysing bootstrapped RA data from the stage 2 GAM. To evaluate long term changes, 2023 median RA was subtracted from baseline median RA for each resample. Hence, a negative value indicated a long-term increase in RA, while a positive value indicated a long-term decline. To test for a significant increase, negative values were assigned to ‘True’ and summed. The total was then divided by 500 (the number of resamples). A result ≥0.95 was required to achieve significance. In the reverse test (decrease), a true value was assigned to positive values. 
For short term changes the same approach was used, only RA changes were evaluated by subtracting 2013 median RA from 2023 median RA. 

Code for determining statistical significance is available [here](/code/analysis/statistically_significant.ipynb).

## 7. License and Attribution Statement
Contains UK Butterfly Monitoring Scheme (UKBMS) data © copyright and database right Butterfly Conservation, the Centre for Ecology & Hydrology, British Trust for Ornithology, and the Joint Nature Conservation Committee.

## 8. References
1.	Hierarchical generalized additive models in ecology: an introduction with mgcv, Smoothing penalties vs shrinkage penalties, A single common smoother plus group-level smoothers with differing wiggliness (Model GI). Pages 16-17.

2.	Hierarchical generalized additive models in ecology: an introduction with mgcv, Smoothing penalties vs shrinkage penalties. Pages 6-7.

3.	Using random effects in GAMs with mgcv, Smooths as random effects.
https://fromthebottomoftheheap.net/2021/02/02/random-effects-in-gams/

4.	The Effect. Chapter 16 – Fixed Effects, Question 2: How do we estimate a regression with individual intercepts?
https://theeffectbook.net/ch-FixedEffects.html

5.	Factor smooth interactions in GAMs, Factor by variables.
https://stat.ethz.ch/R-manual/R-patched/RHOME/library/mgcv/html/factor.smooth.html

6.	Generalized additive modelling and zero inflated count data, Abstract
https://www.sciencedirect.com/science/article/abs/pii/S0304380002001941

7.	Dealing with Varying Detection Probability, Unequal Sample Sizes and Clumped Distributions in Count Data, Introduction
Dealing with Varying Detection Probability, Unequal Sample Sizes and Clumped Distributions in Count Data - PMC

8.	An Overview of Modern Applications of Negative Binomial Modelling in Ecology and Biodiversity, Abstract. 
https://www.mdpi.com/1424-2818/14/5/320

9.	Evaluation of negative binomial and zero-inflated negative binomial models for the analysis of zero-inflated count data: application to the telemedicine for children with medical complexity trial, Conclusions
https://pubmed.ncbi.nlm.nih.gov/37752579/

10.	The consequences of checking for zero-inflation and overdispersion in the analysis of count data, Introduction
https://arxiv.org/pdf/1911.00115

11.	Poisson Regression: A Way to Model Count Data
Poisson Regression: A Way to Model Count Data | DataCamp

12.	Heteroscedasticity in Regression Analysis, Weighted regression
https://statisticsbyjim.com/regression/heteroscedasticity-regression/

13.	Two-sample tests of high-dimensional means for compositional data, 6.1 Analysis of obesity microbiome data, Yuanpei Cao, page 8.

14.	Do not log-transform count data, Results.
https://besjournals.onlinelibrary.wiley.com/doi/10.1111/j.2041-210X.2010.00021.x

15.	Dynamic Generalised Additive Models (DGAM) for forecasting discrete ecological time series, Introduction, Pages 6-7

16.	The UN Environment Programme World Conservation Monitoring Centre, Metadata Factsheet. 5b. Method of Computation. 
https://gbf-indicators.org/metadata/other/A-8-C




