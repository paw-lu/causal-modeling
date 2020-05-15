# 3 Considerations for Large-scale and Network Data

## Table of contents

- [3 Considerations for Large-scale and Network Data](#3-considerations-for-large-scale-and-network-data)
  - [Table of contents](#table-of-contents)
  - [High-dimensional data](#high-dimensional-data)
    - [Example: regularized propensity score](#example-regularized-propensity-score)
  - [Network effects](#network-effects)
    - [Example: identifying peer effects while observational studies is impossible](#example-identifying-peer-effects-while-observational-studies-is-impossible)
    - [Example: use aggregated sub-network](#example-use-aggregated-sub-network)
  - [Data about people](#data-about-people)
    - [Demographic bias](#demographic-bias)
    - [Usage bias](#usage-bias)
    - [Activity bias](#activity-bias)
    - [Preference bias](#preference-bias)

## High-dimensional data

High dimensional data creates estimation problems.
It is sparse.
Examples are
text,
medical.

Solutions:

- Dimensionality reduction (PCA)
- Regularized model (for propensity or regression)
- Transform input space to increase overlap

### Example: regularized propensity score

**What is the effect of the Facebook news feed?**

**Counterfactual:**
Would a user have shared a URL
have they not been exposed to it.

This gives over 3,000 covariates.
Use logistic regression with L2 regularization.

Paradox of measuring text with changing post frequency.

**Should we use word counts
or word probabilities
when comparing text messages from two different groups?**

If group `A` posts more frequently than `B`,
then any word `w` is used more frequently by `A`,
so counts are biased.
Simultaneously, `P_A(w) < P_B(w)`.
Likelihoods are biased as well

Language models are hard to compare when vocabularies differ.
There are some heuristics for smoothing language models,
can use language model over a fixed vocabulary.

## Network effects

Network effects complicate causal inference.
If a person is exposed to some information,
they might share with friends.
Now their friend's outcome may also change.

Breaks the SUTVA assumption.
Consider partitioned sub-networks as a unit of analysis.
Design alternative randomization assignment of estimator.

### Example: identifying peer effects while observational studies is impossible

Consider the problem of separating peer influence from homophliy.

Observed data
is activity of a person `i`
at time `t`
is the same as activity of their friend `j`
at time `t - 1`.

Problem is unobserved traits led to their friendship,
and also to the above common activity.

Without knowing all relevant latent traits,
causal identification is impossible.

### Example: use aggregated sub-network

**Do peers influence us to exercise?**

Instead of individuals,
consider all users in a city as a unit.

Use rainfall to construct a natural experiment on running
and do check to validate IV assumption to the extent possible.

## Data about people

Context is everything

Estimated effect is context-dependent.
May not generalize to other
users,
platform,
or culture.

Common causes of selection bias:

**Structured:**
demographics (gender, age, income)
and usage patterns (number of logins, activity type)

**Unstructured**
Activity (post content, images)
and preferences (items interacted with)

### Demographic bias

Online activity varies by demographics.

Metrics such as time spend on page
are not trustworthy
without controlling for age.

### Usage bias

More activity can simply mean more people are online at the same time,
not due to any specific treatment.

People browse more ad-related products when shown an ad.
But they also browse more of everything.

### Activity bias

Treated and untreated people may differ in many aspects.
People with very different activity content should not be compared.
Match people with similar activity content that is relevant to changes of being treated.

### Preference bias

Any similarity in activity may be due to inherent preferences,
not specified treatment.
Social influence from friends' feeds is most likely over-estimated
because similarity in actions can be homophily.
Vast majority of people's behavior can be predicted by their past actions.
