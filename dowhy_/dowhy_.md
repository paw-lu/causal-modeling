---
jupyter:
  jupytext:
    formats: ipynb,py:percent,md
    text_representation:
      extension: .md
      format_name: markdown
      format_version: '1.2'
      jupytext_version: 1.4.2
  kernelspec:
    display_name: causal
    language: python
    name: causal
---

# DoWhy


## Intro


### Simple start


#### Four steps of causal inference

1. Model a causal problem

DoWhy creates an underlying causal graphical model
for each problem.
You may provide a partial graph,
DoWhy will assume rest of variables
are partial confounders.

2. Identify a target estimand

Based on the causal graph,
DoWhy finds all possible ways
of identifying a desired causal effect
based on the graphical model.

3. Estimate causal effect based on the identified estimand

Dowhy supports methods
based on both back-door criterion
and instrumental variables.
It also provides
a non-parametric permutation test
for testing the statistical significance of obtained estimates.

  - Methods based on estimating the treatment assignment
    1. Propensity-based stratification
    2. Propensity score matching
    3. Inverse propensity weighting
  - Methods based on estimating the response surface
    1. Regression

4. Refute the obtained estimate

Having access to multiple refutation methods
to verify a causal inference
is the key benefit of DoWhy.

Refutation methods are:

  1. Placebo treatment
  2. Irrelevant additional confounder
  3. Subset validation


### DoWhy API

```python
import logging

import dowhy
import dowhy.datasets
import dowhy.plotter
import numpy as np
```

```python
# Make data
data = dowhy.datasets.linear_dataset(
    beta=10,
    num_common_causes=5,
    num_instruments=2,
    num_effect_modifiers=1,
    num_samples=10000,
    treatment_is_binary=True,
)
df = data["df"]
df.head()
```

```python
# Causal graph
data["gml_graph"]
```

```python
for column in ("treatment_name", "outcome_name"):
    print(f"{column}:\t{data[column]}")
```

```python
# Model
model = dowhy.CausalModel(
    data=df,
    treatment=data["treatment_name"],
    outcome=data["outcome_name"],
    graph=data["gml_graph"],
)
model.view_model()
```

```python
# Identify estimand
# This does not depend on the data
identified_estimand = model.identify_effect()
print(identified_estimand)
```

**Backdoor path criterion**
A method of conditioning
where you find the least amount of variables to condition on
that would result in the blocking of all paths
that pass through both the treatment and outcome variables.

**Instrumental variables (IV)**
are variables that can be leveraged
to create natural experiments.

```python
# Estimate the causal effect
# This uses data
causal_estimate = model.estimate_effect(
    identified_estimand, method_name="backdoor.propensity_score_stratification"
)
print(causal_estimate)
print("Causal Estimate is " + str(causal_estimate.value))
```

Now refute.
A **common cause variable**
is on that points
to both the treatment
and the outcome variable
in the causal graph.

```python
# Add random common cause variable
res_random = model.refute_estimate(
    identified_estimand, causal_estimate, method_name="random_common_cause"
)
print(res_random)
```

```python
# Add unobserved common cause variable
res_unobserved = model.refute_estimate(
    identified_estimand,
    causal_estimate,
    method_name="add_unobserved_common_cause",
    confounders_effect_on_treatment="binary_flip",
    confounders_effect_on_outcome="linear",
    effect_strength_on_treatment=0.01,
    effect_strength_on_outcome=0.02,
)
print(res_unobserved)
```

For random placebo variable,
we should expect causal effect
to ideally be zero.

```python
# Replacement treatment with random (placebo) variable
res_placebo = model.refute_estimate(
    identified_estimand,
    causal_estimate,
    method_name="placebo_treatment_refuter",
    placebo_type="permute",
)
print(res_placebo)
```

```python
# Remove a random subset of data
res_subset = model.refute_estimate(
    identified_estimand,
    causal_estimate,
    method_name="data_subset_refuter",
    subset_fraction=0.9,
    random_seed=42,
)
print(res_subset)
```

## Example: Finding causal effects from observed data


Determine whether
the treatment causes the outcome,
or the correlation is due to another cause.


### Data

```python
# Create data
rvar = 1 if np.random.uniform() > 0.5 else 0
data_dict = dowhy.datasets.xy_dataset(10000, effect=rvar, sd_error=0.2)
df = data_dict["df"]
for column in ("treatment_name", "outcome_name", "time_val"):
    print(f"{column}:\t{data_dict[column]}")
df.head()
```

```python
dowhy.plotter.plot_treatment_outcome(
    df[data_dict["treatment_name"]],
    df[data_dict["outcome_name"]],
    df[data_dict["time_val"]],
)
```

### Step 1: Model problem as causal graph

```python
model = dowhy.CausalModel(
    data=df.drop(columns=[""]),
    treatment=data_dict["treatment_name"],
    outcome=data_dict["outcome_name"],
    common_causes=data_dict["common_causes_names"],
    instruments=data_dict["instrument_names"],
)
model.view_model(layout="dot")
```

### Step 2: Identify causal effect using properties of graph

```python
identified_estimand = model.identify_effect()
print(identified_estimand)
```

### Step 3: Estimate causal effect

```python
estimate = model.estimate_effect(
    identified_estimand, method_name="backdoor.linear_regression"
)
print("Causal Estimate is " + str(estimate.value))

# Plot Slope of line between treamtent and outcome =causal effect
dowhy.plotter.plot_causal_effect(
    estimate, df[data_dict["treatment_name"]], df[data_dict["outcome_name"]]
)
```

```python
# Check if estimate is correct
print("DoWhy estimate is " + str(estimate.value))
print("Actual true causal effect was {0}".format(rvar))
```

### Step 4: Refute the estimate

```python
# Add a random common cause variable
res_random = model.refute_estimate(
    identified_estimand, estimate, method_name="random_common_cause"
)
print(res_random)
```

```python
# Replace treatment with random placebo variable
res_placebo = model.refute_estimate(
    identified_estimand,
    estimate,
    method_name="placebo_treatment_refuter",
    placebo_type="permute",
)
print(res_placebo)
```

```python
# Remove a random subset of the data
res_subset = model.refute_estimate(
    identified_estimand,
    estimate,
    method_name="data_subset_refuter",
    subset_fraction=0.9,
)
print(res_subset)
```

## Different estimation methods for causal inference


### Data

```python
data = dowhy.datasets.linear_dataset(
    beta=10,
    num_common_causes=5,
    num_instruments=2,
    num_treatments=1,
    num_samples=10000,
    treatment_is_binary=True,
    outcome_is_binary=False,
)
df = data["df"]
df.head()
```

### Identify the causal estimand

```python
# With graph
model = dowhy.CausalModel(
    data=df,
    treatment=data["treatment_name"],
    outcome=data["outcome_name"],
    graph=data["gml_graph"],
    instruments=data["instrument_names"],
    logging_level=logging.INFO,
)
model.view_model()
```

```python
identified_estimand = model.identify_effect()
print(identified_estimand)
```

### Linear regression


Fit a linear regression model
to the treatment and identified confounders variables.
The coefficient of the treatment variable
is extracted as the treatment effect.

```python
causal_estimate_reg = model.estimate_effect(
    identified_estimand,
    method_name="backdoor.linear_regression",
    test_significance=True,
)
print(causal_estimate_reg)
print("Causal Estimate is " + str(causal_estimate_reg.value))
```

### Stratification


Stratify users into bins
based on identified confounders.

```python
causal_estimate_strat = model.estimate_effect(
    identified_estimand,
    method_name="backdoor.propensity_score_stratification",
    target_units="att",
)
print(causal_estimate_strat)
print("Causal Estimate is " + str(causal_estimate_strat.value))
```

### Propensity matching


Model a user's likelihood
to receive the treatment.
Match individual users with equal propensity scores
and compare treatment effect.

```python
causal_estimate_match = model.estimate_effect(
    identified_estimand,
    method_name="backdoor.propensity_score_matching",
    target_units="atc",
)
print(causal_estimate_match)
print("Causal Estimate is " + str(causal_estimate_match.value))
```

### Weighting


Use inverse propensity score
to assign weights to units in data.
Find difference in weighed sum
between treated and untreated populations.

```python
causal_estimate_ipw = model.estimate_effect(
    identified_estimand,
    method_name="backdoor.propensity_score_weighting",
    target_units="ate",
    method_params={"weighting_scheme": "ips_weight"},
)
print(causal_estimate_ipw)
print("Causal Estimate is " + str(causal_estimate_ipw.value))
```

### Instrumental variable


Use instrumental variable method
to compute effect of treatment.
Specify which instrumental variable to use with `method_params`.
Look for variations in instrumental variable
due to external effects.
Like the treatment being on the Oprah Winfrey show,
or featured on the front of the app page.
Treat difference in effect as causal estimate.

```python
causal_estimate_iv = model.estimate_effect(
    identified_estimand,
    method_name="iv.instrumental_variable",
    method_params={"iv_instrument_name": "Z0"},
)
print(causal_estimate_iv)
print("Causal Estimate is " + str(causal_estimate_iv.value))
```

### Regression discontinuity


Compare uses
due to arbitrary changes in treatment.
Example is finding how effective
an app store recommender is.
The recommendation system
will typically show user top 3 highest rated recommendations.
Can compare performance of /#3 to /#4—
which is not shown to the user,
but is similarly scored to /#3.

```python
causal_estimate_regdist = model.estimate_effect(
    identified_estimand,
    method_name="iv.regression_discontinuity",
    method_params={
        "rd_variable_name": "Z1",
        "rd_threshold_value": 0.5,
        "rd_bandwidth": 0.1,
    },
)
print(causal_estimate_regdist)
print("Causal Estimate is " + str(causal_estimate_regdist.value))
```

```python

```
