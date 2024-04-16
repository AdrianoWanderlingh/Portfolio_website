# Building a Robust qPCR Data Processing Pipeline

High-throughput quantitative Polymerase Chain Reaction (qPCR) is a pivotal technique for analyzing gene expression when a large number of biological samples have to be processed. The analysis of these expression datasets, however, is made difficult by the large proportion of missing data, particularly data that is Missing Not At Random (MNAR). Such missing data increase as gene expression levels (measured in Cycle Threshold, or Ct values) decrease, which can compromise the  reliability of the statistical analysis.

This document explains a qPCR data processing method designed to handle these missing data issues, ensuring accurate studies of gene expression.

## Introduction
Quantitative Polymerase Chain Reaction (qPCR) is a widely used technique in molecular biology that quantifies DNA or RNA in a sample. Understanding its results requires accounting for missing data points, which often occur when the data is below the detection limit of the qPCR machines. This situation is problematic because it can distort the interpretation of gene expression levels.

### Understanding the Basics
- __qPCR__: A method used to amplify and quantify a targeted DNA molecule.
- __Cycle Threshold (Ct)__: The number of cycles needed until the signal is strong enough to be detected. Lower Ct values indicate higher gene expression.
- __Missing Not At Random (MNAR)__: Missing data with a non random distribution in the dataset due to reasons that relate to the nature of the data itself. In this case, the low gene expression levels cause large proportion of data to be close or over the Limit of Detection (LOD) of the qPCR machines.

<p>
    <img src="images/missing_vs_Ct.png?raw=true" alt="Proportion of missing data">
    <br>
    <em>Scatter plot showing the proportion of missing data against mean gene expression (mean Cycle Threshold, or Ct). This illustrates that genes expressed at lower levels - high Cts - are more likely to be undetected, highlighting the challenge of MNAR data in qPCR.</em>
</p>

## Pipeline Overview
This data processing pipeline consists of several steps from initial sample handling to advanced statistical analysis, ensuring high integrity and reliability of the results.

### 1. Initial Quality Control
**Objective:** To ensure all samples meet basic quality standards before further analysis.
- **Housekeeping Gene Verification**:Verifies stability in the expression of the housekeeping gene _EF-1α_. Inconsistent or missing housekeeping gene data leads to exclusion of the sample to maintain integrity.
- **Technical Error Adjustments**: Includes corrections for technical errors (T.E.) during sample handling that might affect data quality, such as pipetting errors or cross-contamination.

### 2. Gene of Interest Checks
**Objective:** To ensure the integrity of data for genes critical to our study, namely _EF-1α_, _Defensin_, _Hymenoptaecin_, and _PPO_.
- We conduct targeted checks as shown in the flow diagram, which are based  on peer-reviewed scientific literature and data characteristics.


<p>
    <img src="images/immune_pipeline.png?raw=true" alt="Data processing pipeline">
    <br>
    <em>Flow diagram of the qPCR data pre-processing pipeline detailing each stage, from initial quality control through advanced handling of missing values to gene-specific integrity checks, ensuring robust data validation. Highlighted in red, the resulting data treatment depending on the satisfaction of the conditions. LOD: limit of detection; T.E.: technical error; Diff.Ct: difference in Ct values of the duplicate.</em>
</p>

The pre-processing of the data allows the categorisation of the samples into groups determinng if the samples are reliable, unreliable or to be imputed for subsequent statistical tests. This ensures unbiased and robust analyses of the analysed qPCR samples.

| Category                 | _defensin_ | _hymenoptaecin_ | _PPO_  |
|--------------------------|------------|-----------------|--------|
| **Valid**                | 614        | 185             | 328    |
| **Invalid**              | 176        | 167             | 171    |
| **Discard NA Ct**        | 2          | 38              | 44     |
| **Impute (>DT)**         | 0          | 2               | 1      |
| **Impute (>LOD-T.E.)**   | 0          | 169             | 115    |
| **Impute (2NA)**         | 0          | 231             | 133    |


### 3. Handling and Imputation of Missing Data
**Objective:** To accurately estimate missing qPCR data points using robust statistical methods.
- **Quantile Regression Imputation for Left-Censored Data (QRILC)**: Estimates the upper boundary of missing data using quantile regression, suitable for left-censored data where measurements fall below detection limits.
- **Half-Minimum Method (HM)**: Assigns the median of observed minimal values to missing data, minimizing data distortion.

#### checking the Imputation Reliability
The effectiveness of the imputation methods was validated by conducting an in-depth comparison between several truncated and then re-imputed versions of the _defensin_ dataset  and the original full dataset itself. the _defensin_ gene was selected as it was the only that did not require imputation. 15 truncated-imputed datasets were simulated via logistic regression to mimic various missing data scenarios, with increasing proportions of missing data and performing imputation with the above mentioned techniques. The comparison highlights the superior performance of the HM method in maintaining the directionality and consistency of effects and significance levels observed in the original dataset, as shown in the plot below.

<p>
    <img src="images/immune_imputation_method.png?raw=true" alt="Imputation methods comparison">
    <br>
    <em>Summary of the reliability of the imputation method showing the effects of (A) the ant task group and (B) of the group size on the measured relative expression of defensin. The scatterplots illustrate the relationship between model estimates (x-axis) and p-values (y-axis) for each one of the 15 truncate-imputed datasets.  E.S.: Effect size. Different imputation methods are represented by distinct shapes. The x marks highlight the values for the original Defensin dataset. The red dashed line shows the threshold significance level α = 0.05. The black line is centred on model estimates equal to zero. Marginal histograms are displayed at the top and right edges of the scatterplot, representing the distribution of model estimates (top) and p-values (right).</em>
</p>

The performance of both imputation methods was evaluated by comparing the real Ct values for the _defensin_ dataset that have been censored with the values imputed in their place. To do so, the Normalized Root Mean Squared Error was used, as it is a widely employed metric for assessing the effectiveness of imputation techniques. As the proportion of missing data increases in the truncated-imputed datasets, HM imputation shows a lower increase in NRMSE values than QRILC

<p>
    <img src="images/imputation_comparison.png?raw=true" alt="Imputation method evaluation">
    <br>
    <em>Comparison of the performance of three imputation methods, Half-minimum (HM), Quantile Regression Imputation for Left-Censored data (QRILC) and zero replacement, in terms of the ranked Root Mean Squared Deviation (NRMSE). The x-axis represents the proportion of simulated missing data, which corresponds to the degree of left-censoring. The y-axis represents the mean NRMSE rank for all tested steepness value k. Dashed lines represent the proportion of missing data for the two imputed genes.</em>
</p>

### Assessing the Pipeline Stability
We assessed the stability of our pipeline under various experimental conditions by adjusting detection limits (LOD) and technical error thresholds (T.E.). The stability analysis confirms that our pipeline delivers reliable and consistent results, even under varying LOD and TE settings, showcasing minimal fluctuations in p-values and effect sizes, crucial for dependable gene expression analysis.

<p>
    <img src="images/immune_pipeline_stability.png?raw=true" alt="Pipeline stability analysis">
    <br>
    <em>Summary of the pipeline stability testing showing the effects of (A) the ant task group and (B) of the group size on the measured relative expression of defensin, hymenoptaecin and PO. The scatterplots illustrate the relationship between model estimates (x-axis) and p-values (y-axis) for each iteration of the pipeline, with each gene represented by a distinct colour. E.S.: Effect size. Different imputation methods are represented by distinct shapes. The x marks highlight the combination of pipeline thresholds that are shown in the statistical analysis. The red dashed line shows the threshold significance level α = 0.05. The black line represents model estimates equal to zero. Marginal histograms are displayed at the top and right edges of the scatterplot, representing the distribution of model estimates (top) and p-values (right) separately for each gene. Each histogram is colour-coded to match its corresponding gene in the scatterplot. </em>
</p>

## Final considerations

This project addresses the pervasive issue of MNAR data in high-throughput qPCR studies and sets a benchmark for data handling that enhances the reliability and reproducibility of gene expression analysis. By applying rigorous data processing and imputation methods, this pipeline allows for robust statistical analysis, serving as a model for addressing similar challenges in the field of molecular biology.

## References
- De Ronde, M. W. J., Ruijter, J. M., Lanfear, D., Bayes-Genis, A., Kok, M. G. M., Creemers, E. E., Pinto, Y. M., & Pinto-Sietsma, S. J. (2017). Practical data handling pipeline improves performance of qPCR-based circulating miRNA measurements. RNA, 23(5), 811–821. https://doi.org/10.1261/RNA.059063.116
- Wei, R., Wang, J., Su, M., Jia, E., Chen, S., Chen, T., & Ni, Y. (2018). Missing Value Imputation Approach for Mass Spectrometry-based Metabolomics Data. Scientific Reports 2018 8:1, 8(1), 1–10. https://doi.org/10.1038/s41598-017-19120-0
- McCall, M. N., McMurray, H. R., Land, H., & Almudevar, A. (2014). On non-detects in qPCR data. Bioinformatics, 30(16), 2310–2316. https://doi.org/10.1093/BIOINFORMATICS/BTU239

## Repository Link
All project codes and datasets are available in my [GitHub repository](https://github.com/AdrianoWanderlingh/Ant_Tracking/tree/main/Scripts/PhD-Ant_Colonies_Tracking_Analysis/molecular_bio_analysis).
