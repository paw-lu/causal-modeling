# 4 High-level Awareness of Broader Landscape in Causal Reasoning

## Table of contents

- [4 High-level Awareness of Broader Landscape in Causal Reasoning](#4-high-level-awareness-of-broader-landscape-in-causal-reasoning)
  - [Table of contents](#table-of-contents)
  - [Outline](#outline)
  - [Causal discovery](#causal-discovery)
  - [Heterogenous treatment effects](#heterogenous-treatment-effects)
  - [Machine learning and causal inference](#machine-learning-and-causal-inference)
  - [Reinforcement learning and causal inference](#reinforcement-learning-and-causal-inference)
    - [Example: Contextual bandits on Yahoo! news](#example-contextual-bandits-on-yahoo-news)
  - [Combine techniques](#combine-techniques)
    - [Example: randomization + intrumental variable](#example-randomization--intrumental-variable)
  - [Conclusion: Best practices](#conclusion-best-practices)

## Outline

- Discovery of causal relationships from data
- Heterogeneous treatment effects
- Machine learning, representations and causal inference
- Reinforcement learning and causal inference
- "Automated" causal inference

## Causal discovery

We have discussed causal inference: effects of causes.

But a complementary question is causal discovery.
Harder problem.

## Heterogenous treatment effects

Average causal effect does not capture individual-level variations.
Stratification is one of the simplest methods
for heterogeneous treatment by strata.
Typical strata are demographics.
Use ML methods like random forest for high-dimension.

## Machine learning and causal inference

Supervised ML predicts value under the training distribution `P(X, y)`.
Causal inference predicts value under the counterfactual distribution `P'(X, y)`.

Predicting the counterfactual causal inference.

Predicting individual treatment effects can be considered domain adaptation.
User regularization and transformation of input features.

Generalizing predictions to new domains.

Machine learning use causal inference for robust, generalizable prediction.
Causal inference use ML algorithms to better model the non-linear effect of confounders,
or find low-dimensional representations.

## Reinforcement learning and causal inference

Multi-armed bandits.
Two goals—
show the best known algorithm to most users,
keep randomizing to update knowledge about competing algorithm.

Algorithm:
`e`-greedy muli-armed bandit.
Repeat this:

- With low probability `e`,
  choose an output item randomly
- Otherwise,
  show the current-best algorithm.

### Example: Contextual bandits on Yahoo! news

Different news article to display.
Randomized the articles show using `e`-greedy policy.
Use context of visitor—
user, browser, time
—to have different current-best algorithms for different contexts

## Combine techniques

### Example: randomization + intrumental variable

Treatment example:
You cannot randomize who exercises,
but you _can_ provide incentives to who joins the gym.

Algorithm example:
You cannot remove recommendations at random,
but _could_ advertise a focal product to a random subset of people on the homepage.

## Conclusion: Best practices

- Always follow the four steps:
  - Model
  - Identify
  - Estimate
  - Refute (the most important)
- Aim for simplicity
  - If analysis too complicated,
    likely wrong
- Try at least two methods with different assumptions
  - Higher confidence in estimate if both methods agree
