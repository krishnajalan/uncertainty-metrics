# Bayesian Expected Calibration Error (BECE)

This document provides further details for the Bayesian expected calibration
error statistic. This document is intended for people who are familiar with
Bayesian modeling who want to understand the internals of the
`predictive_metrics.calibration.bayesian_expected_calibration_error` method.

The user-facing documentation is at [available here](index.md#bayesian-ece).

[TOC]

## Generative Model

![drawing](https://docs.google.com/drawings/d/1w4GFeDRi0aIYgcalPy1QFguBe7Je1C6w2_Fx3VBMXVE/export/png)

The model is given by the following generative mechanism.

$$q \sim \textrm{Dirichlet}(\alpha \, \mathbb{1}_{2\times M}),$$

The distribution $$q$$ is over $$2 \times M$$ outcomes, where the first $$M$$
outcomes mean $$z_i = 0$$ and the second set of $$M$$ outcomes mean $$z_i = 1$$.
Thus,

$$(z_i,b_i) \sim \textrm{Categorical}(q), \quad i=1,\dots,n,$$

The per-bin means are generated by a
[truncated Normal distribution](https://en.wikipedia.org/wiki/Truncated_normal_distribution),

$$p_i \sim \textrm{TruncatedNormal}(\mu_{b_i}, \sigma^2, I_{b_i}),
\quad i=1,\dots,n.$$

## Inference

For inference we are given a list of $$(\hat{y}_i,y_i,p_i)$$ triples, where:

*   $$\hat{y}_i$$ is the decision label of the prediction system, typically
    being the $$\textrm{argmax}$$ over the predicted probabilities;
*   $$y_i$$ is the true observed label; and
*   $$p_i$$ is the probability $$p(\hat{y}_i|x_i)$$ provided by the prediction
    system.

Each probability $$p_i$$ can be uniquely assigned to a bin $$b_i = b(p_i)$$, and
each pair $$(\hat{y}_i,y)$$ can be uniquely assigned to $$z_i =
\mathbf{1}_{\{\hat{y}_i = y\}}$$.

The posterior $$p(\alpha,\mu|Z,B,P)$$ factorizes as

$$p(q,\mu|Z,B,P) = p(q|Z,B) \, p(\mu|P,B).$$

The first factor $$p(q|Z,B)$$ is a Dirichlet with analytic solution in the
Dirichlet-Multinomial model. The second factor has an
[analytic posterior in the Normal model](https://www.cs.ubc.ca/~murphyk/Papers/bayesGauss.pdf),
and the truncation does not affect this because it can be seen as a product with
an indicator function over the respective probability interval $$I_b \subset
[0,1]$$.

Given the posterior $$p(q,\mu|Z,B,P)$$ we draw samples $$(q,p)$$ and compute the
analytic ECE formula for each sample,

$$ECE(q,p) := \sum_{m=1}^M q_{1,m} |q_{1,m} - p_m|.$$