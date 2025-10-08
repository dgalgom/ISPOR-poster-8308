# ISPOR-poster-8308
<img width="1600" height="900" alt="ISPOR_icon" src="https://github.com/user-attachments/assets/5ff00617-d8d9-44b5-af62-d43de09e00f1" />

## From complex statistics to clinically meaningful insights: Interpreting results in meta-analyses of continuous outcomes
### 📋 Overview
This repository provides a comprehensive methodological framework for translating abstract statistical metrics—specifically Standardized Mean Differences (SMDs)—into clinically interpretable, scale-specific probabilities of achieving meaningful patient benefit.

**The Core Challenge**:
Meta-analyses often report treatment effects in standardized units (e.g., SMD = -0.60 standard deviations), which are:

  + ❌ Difficult for clinicians to interpret
  + ❌ Abstract for patients to understand
  + ❌ Disconnected from established clinical thresholds
  + ❌ Focused on statistical significance rather than clinical meaningfulness

**Our Solution**:
A probabilistic framework that:

  + ✅ Translates SMDs into scale-specific units (e.g., points on a 0-10 pain scale)
  + ✅ Compares effects against evidence-based clinical thresholds (e.g., Minimal Clinically Important Difference)
  + ✅ Quantifies the probability of achieving clinically meaningful benefit
  + ✅ Accounts for uncertainty in both treatment effects and thresholds
  + ✅ Provides robust sensitivity analyses across distributional assumptions


### 🎯 Key Features
| Feature | Description |
|---------|-------------|
|📊 SMD Translation | Converts standardized effect sizes to scale-specific units using external SD references |
|🎲 Probabilistic Assessment | Estimates probability of exceeding clinical significance thresholds via Monte Carlo simulation |
|🔬 Dual Comparison Methods | Distributional MCID (realistic) + Fixed threshold (traditional) approaches |
|📈 Uncertainty Quantification | 95% credible intervals via 1,000 iterations accounting for parameter uncertainty|
|🔄 Sensitivity Analyses | Tests robustness across Beta/Skew-normal distributions and varying CV assumptions (20%-40%)|

### 🧩 The Framework: From Statistics to Clinical Insights
*Step 1: The Gap Between Statistical Significance and Patient-Centered Outcomes*
Traditional clinical trial reporting: `Treatment A reduced pain with a SMD = -0.60 SDs (95% CI -0.86 to -0.34; p-value < 0.01)`

**What clinicians/patients need to know**:

  + How many points on the pain scale will patients improve?
  + What's the probability this improvement is meaningful to patients?
  + How does this compare to the minimal change patients care about?

**The Problem**:
Statistical significance (p < 0.001) tells us the effect is real but not whether it's clinically important. An SMD of -0.60 could represent a trivial or substantial benefit depending on the outcome scale—context is lost in standardization.

*Step 2: Translation to Scale-Specific Units*

We convert abstract SMDs to interpretable, scale-specific metrics:

```{r}
# Input: Meta-analysis result
SMD <- -0.60
CI_lower <- -0.86
CI_upper <- -0.34

# External reference: Pooled SD from target scale
# (e.g., chronic pain studies using 0-10 Numeric Rating Scale)
SD_ref <- 2.25  # From published literature

# Translation formula
Effect_points <- SMD × pooled_SD
Effect_points  # = -1.35 points on 0-10 NRS

# Standard error calculation
SE_SMD <- (CI_upper - CI_lower) / 3.92
SE_points <- SE_SMD × SD_ref
SE_points  # = 0.30 points
```

**Output**:
Treatment reduces pain by 1.35 points (SE = 0.30) on the 0-10 NRS

  + ✅ Now clinicians understand the magnitude in familiar units
  + ✅ Patients can contextualize expected benefit
  + ✅ Ready for comparison against clinical thresholds

*Step 3: Probabilistic Comparison Against Clinical Thresholds*

**3a. Defining the Clinical Threshold**

The Minimal Clinically Important Difference (MCID) represents the smallest change patients perceive as beneficial.
For chronic pain on the NRS (0-10):

  + MCID point estimate: -2.00 points (Farrar et al., 2001; Dworkin et al., 2008)
  + Heterogeneity: Varies by population, baseline severity, and methods
  + Coefficient of Variation (CV): 20% (primary) to 40% (sensitivity) based on systematic reviews

**3b. Primary Analysis: Distributional MCID Comparison**

Rationale: Both the treatment effect AND the MCID contain uncertainty. Rigorous assessment must account for both.
```{r}
# Treatment Effect Distribution
effect_mean <- -1.35
effect_SE <- 0.30
effect_distribution <- Normal(μ = -1.35, σ = 0.30)

# MCID Distribution (accounting for threshold uncertainty)
mcid_mean <- -2.00
mcid_CV <- 0.20  # 20% coefficient of variation
mcid_SD <- abs(mcid_mean) × mcid_CV  # = 0.40
mcid_distribution <- Normal(μ = -2.00, σ = 0.40)

# Monte Carlo Simulation
set.seed(123)
n_sims <- 10000

effect_draws <- rnorm(n_sims, mean = effect_mean, sd = effect_SE)
mcid_draws <- rnorm(n_sims, mean = mcid_mean, sd = mcid_SD)

# Probability of Clinical Significance
prob_clinical_sig <- mean(effect_draws <= mcid_draws)

# Result: 13.33%
```

**Interpretation**:
There is a 13.33% probability that the treatment effect achieves or exceeds the MCID when accounting for uncertainty in both quantities.

**3c. Secondary Analysis: Fixed MCID Threshold**

Rationale: Traditional approach for comparison. Treats MCID as a known constant (less realistic but commonly used).
```{r}
# Compare effect distribution to single MCID value
mcid_fixed <- -2.00

prob_fixed <- mean(effect_draws <= mcid_fixed)

# Result: 1.90%
```

Comparison:

  + Distributional MCID: 13.33% (conservative, accounts for threshold uncertainty)
  + Fixed MCID: 1.90% (optimistic, ignores threshold uncertainty)
  + *Difference*: 11.43 percentage points

**Recommendation**: Use distributional approach for primary analysis as it provides more realistic, defensible estimates.

**3d. Quantifying Uncertainty: Credible Intervals for Probabilities**

The Question: We have a point estimate (13.33%), but what's the uncertainty around that probability itself?
Solution: Account for parameter uncertainty in the treatment effect estimate by resampling.

```{r}
# Uncertainty propagation via parametric bootstrap
n_iterations <- 1000
prob_distribution <- numeric(n_iterations)

for(i in 1:n_iterations) {
  # Sample "true" treatment effect accounting for sampling uncertainty
  sampled_effect_mean <- rnorm(1, mean = effect_mean, sd = effect_SE)
  
  # Generate draws from this sampled parameter
  effect_draws_i <- rnorm(n_sims, mean = sampled_effect_mean, sd = effect_SE)
  mcid_draws_i <- rnorm(n_sims, mean = mcid_mean, sd = mcid_SD)
  
  # Calculate probability for this iteration
  prob_distribution[i] <- mean(effect_draws_i <= mcid_draws_i)
}

# 95% Credible Interval
prob_mean <- mean(prob_distribution)
prob_CI <- quantile(prob_distribution, c(0.025, 0.975))

# Result: 13.33% (95% CrI: 0.69%, 45.32%)
```
<img width="2550" height="2100" alt="dist_primary" src="https://github.com/user-attachments/assets/17039027-64fb-49f0-935f-b072c7726f75" />

**Output**:
**Probability of clinical significance: 13.33% (95% CrI: 0.69%, 45.32%)**

  + ✅ Provides complete uncertainty quantification
  + ✅ Reflects finite sample size and estimation error
  + ✅ Enables probabilistic decision-making with known confidence bounds

## Visualization of the generated distributional random draws and clinical probabilities

It is easier to understand this process by visualizing it directly. Basically, what we did firstly is to generate treatment effect and MCID distributions using the re-expressed meta-analytic point estimate and SD, and MCID and SD (CV=20%). Then, we compared the generated 10,000 random draws (plot on the left). However, to account for parameter uncertainty, we iterated this process 1,000 times using a random draw for the mean treatment effect to generate those 10,000 distributional values and compare to the MCID values. In the second plot, an example of this process applying a mean treatment effect of -1.00 point was generated.


<img width="4200" height="1800" alt="comparison_draws" src="https://github.com/user-attachments/assets/9d2dcd97-df1c-4816-9206-3bc4b2a8268f" />

After 1,000 iterations, we have a collection of mean probabilities of the treatment being clinically important for the patients, which were distributed as follows:

<img width="2400" height="1800" alt="prob_dist" src="https://github.com/user-attachments/assets/a1003458-8b75-414c-9573-ed523ee29f58" />

**Step 4: Robustness Through Sensitivity Analyses**

To ensure conclusions aren't artifacts of specific assumptions, we tested:

*4a. Alternative Statistical Distributions*

Rationale: Pain scales are bounded (0-10), and patient responses may be asymmetric. Normal distributions may not fully capture these properties.
Sensitivity Analysis: *Beta Distribution*.
Clinical Justification:

  + Pain scales have natural bounds (0-10)
  + Beta distributions respect these boundaries
  + Evidence suggests pain outcomes can show floor/ceiling effects
  + Used in Bayesian pain trial designs (Berry et al., 2004)

```{r}
# Convert treatment effect to Beta distribution
# Rescale to [0,1], transform to beta parameters, generate draws

mean_01 <- 1.35 / 10  # Effect as improvement on 0-10 scale
sd_01 <- 0.30 / 10
var_01 <- sd_01^2

# Method of moments for alpha/beta parameters
alpha <- mean_01 * (mean_01 * (1 - mean_01) / var_01 - 1)
beta_param <- (1 - mean_01) * (mean_01 * (1 - mean_01) / var_01 - 1)

# Generate Beta draws and back-transform
beta_draws_01 <- rbeta(n_sims, shape1 = alpha, shape2 = beta_param)
effect_beta <- -(beta_draws_01 * 10)  # Convert back to pain reduction

# Compare to MCID
prob_beta <- mean(effect_beta <= mcid_draws)

# Result: 13.47% (95% CrI: 0.82%, 43.59%)
```

<img width="2550" height="2100" alt="dist_sens_1" src="https://github.com/user-attachments/assets/5e4e5f1e-07cf-4109-8863-950d51b1a27e" />

**Finding**: Beta distribution yields 13.47%—consistent with primary analysis (13.33%), differing by only ~0.1 percentage points.

*4b. Varying MCID Uncertainty (Coefficient of Variation)*

Rationale: Published MCID estimates show substantial heterogeneity across:

  + Different populations (neuropathic vs. musculoskeletal pain)
  + Baseline pain severity (mild vs. severe)
  + Anchor methods (patient global impression vs. functional improvement)
  + Study designs (within vs. between-subject)

Primary Analysis CV = 20%: Moderate uncertainty, mid-range of literature estimates
Sensitivity Analysis: CV = 40%: Upper bound representing maximum plausible real-world variation

```{r}
# High MCID uncertainty scenario
mcid_CV_high <- 0.40
mcid_SD_high <- abs(mcid_mean) * mcid_CV_high  # = 0.80

mcid_draws_high <- rnorm(n_sims, mean = mcid_mean, sd = mcid_SD_high)

prob_high_CV <- mean(effect_draws <= mcid_draws_high)

# Result: 22.41% (95% CrI: 6.21%, 47.19%)
```

<img width="2550" height="2100" alt="dist_sens_2" src="https://github.com/user-attachments/assets/ff375bc5-6777-4af8-b13d-99238f5e3524" />

**Finding**: Higher uncertainty increases probability to 22.41% (+10 percentage points from primary), but still remains <30%, indicating conclusions are robust even under pessimistic assumptions about MCID heterogeneity.
