# Project Methodology
## Contents

1. Key Terms
	1. Baseline Index
	2. Relative Abundance / Index Ratio
	3. Site/Species combinations
2.	Model Selection
	1. Model Structure
	2. Model Distribution
	  	- Profiling the Data
	  	- Selecting a distribution
3.	Stage 1 GAM
	1. Data Requirements
  	2. Model Optimisation 	
4.	Stage 2 GAM
	1. Data Requirements
  	2. Model Optimisation 	
5.	Bootstrapping
6.	Statistical Significance
7.	References

## 1. Key Terms
### i. Baseline Index
This is the count recorded in the baseline year (1993) for a single species at a single site.

### ii. Relative Abundance / Index Ratio
Relative Abundance (RA) and index ratio have the same meaning and are used interchangeably through the report. They describe the increase or decrease of a species count at a site in a specific year, relative to its baseline year. For example, if species code A received 8 counts at site code 1 in 1993, but 20 counts from the same site in 1997, the relative abundance for 1997 would be 2.5. Similarly, if in 2005, only 2 counts were recorded, the relative abundance for that year would be 0.25. This way a relative decrease is represented by values less than 1, and an increase by values greater than 1. The formula is shown below:

![Alt text](/images/methodology/ra_equation.png)

For small sample sizes RA is not very useful for summarising national trends. Natural stochasticity makes it difficult to identify the true signal, and location can also have a significant effect. However, when RA is combined over many different sites, it is possible to detect trends in some species. 

### iii. Site/species combination
Refers to a specific ‘survey grouping’ (a single species from a single site). For example, if 20 species are observed at site A, then 20 site/species combinations exist at this site. Similarly, if 20 of the same species are observed at sites A and B, then 40 site/species combinations exist across these two sites.

## 2. Model Selection
### i. Model Structure
GAMs were selected for their ability to capture non-linear trends - a characteristic associated with species abundance time-series data. Two explanatory variables were considered: time in years (assumed to be continuous for modelling); and species (categorical). The desired response variable was RA.

One issue to arise from this set up, was that many site/species combinations were not surveyed in the baseline year, limiting the sample size of RA training data for each species. This was a key consideration when deciding upon a suitable GAM structure. Several well-recognised options are evaluated in table 1.  


![Alt text](/images/methodology/gam_structure_table.png)

Summarising table 1:
- Hierarchical GAMs, while preferred for their data sharing capabilities, were not feasible in Python.
- A fixed effect GAM would be ineffective for analysing species-specific trends.
- Both factor smooth models exhibited limited data sharing capabilities meaning a large proportion of survey data would be useless. Additionally, the sharing of hyperparameters in these last two models ignored the requirement for variable smooth complexity.

It was decided that two single-term-GAMs would be implemented. A ‘stage 1 GAM’ for predicting missing baseline counts, and a ‘stage 2 GAM’ for relative abundance trends. In both models the explanatory variable would be time (in years). Prediction of missing baseline counts would enable the calculation of RA for more site/species combinations, reducing the need for data sharing, a limitation of the factor smooth models. 

## 3. Model Distribution
### i. Profiling the Data
Profiling UKBMS count data for all site/species combinations was not possible because absolute index scores from different groups were incomparable. Relative abundance profiling was possible, but baseline records were not always present, and this method ignored count-level distribution. From other ecological abundance studies, it was noted that zero inflation, right skew, over-dispersion (a term used when variance exceeds the mean) and heteroskedasticity (variance increases with the mean) were characteristics typical of this data type [6], [7]. Another important, yet obvious observation was that count data cannot be negative. 
### ii. Selecting a Model
The negative binomial model appeared the most compatible. This model type is preferred in ecological count studies for its ability to fit datasets characterised by overdispersion and zero inflation [8], [9]. Unfortunately, this distribution was not available in Pygam. 

The Poisson model was available and stood out for its ability to predict integers that were non-negative [10]. However, Campbell et al. noted the potential for ‘serious errors’ in over-dispersed, zero-inflated datasets [11]. 

The Gaussian model seemed a poor fit considering its assumptions (normal distribution of residuals, and constant variance across a fitted range). In practice, when fitted to a heteroskedastic dataset, gaussian models fail to recognise noise at increased means, developing overly complex curves in these areas to minimise the sum of the squares. Squaring of the residuals, additionally has the effect of allowing large values to disproportionately influence the resulting smooth [12].  
A solution was to log transform count data using log(x+ɛ) where ɛ, a small constant, was set to 1. This approach, similar to that used by Cao et al [13], enabled transformation of zero count data. The effects of log transformation were three-fold:
- Multiplicative differences between counts became additive. This ‘compressed’ the dataset (large values in particular) reducing right skew.
- The variance of counts reduced.
- Variance became more constant across the fitted range.

Better compliance with the Gaussian assumptions meant the data could be modelled after transformation. This method was approved for the stage 1 GAM (count data) and the stage 2 GAM (RA). 

To minimise bias and error associated with log transformation, species subsets were omitted if zero counts, exceeded 15%. This is approximately the threshold observed by B. O’Hara et al, below which the log(x+1) transformation exhibited the least error and bias when compared with other transformations in generalised linear models [14].  

