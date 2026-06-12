---
layout: post
title:  "Improving fitlm function and expanding LinearModel constructor"
date:   2026-06-11
description: "This week the LinearModel finally came to life. fitlm works end to end, the constructor is fully populated, and both mentors gave feedback that shaped how I'll write tests and look at performance going forward."
---

<style>
  .postTitle {
    width: 100% !important;
    max-width: 100% !important;
    font-size: 1.7em !important;
  }
  @media (min-width: 800px) {
    .postTitle {
      font-size: 2.1em !important;
      width: 100% !important;
    }
  }
</style>

<p class="intro"><span class="dropcap">I</span>f you have been following along, you know I have been building a <a href="https://in.mathworks.com/help/stats/linearmodel.html">LinearModel</a> class for GNU Octave as part of Google Summer of Code 2026. The goal of the whole <a href="https://summerofcode.withgoogle.com/programs/2026/projects/YKFoacXh">project</a> is simple: when someone runs a linear regression in Octave, instead of getting back a bunch of raw numbers, they get back a smart object that holds everything, including the fitted equation, the error statistics, diagnostic checks, and tools to make predictions. Think of it like getting a full lab report instead of just the answer to one question.</p>

## Project Status

To give a bit of context before jumping in, here is where things stand after three weeks.

[**Week 1**]({{ '/blog/introduction-and-gsoc-timeline/' | relative_url }}) was about setting up the project, reading through the MATLAB documentation, planning the architecture, and creating the directory structure and stub files in the statistics package repository.

[**Week 2**]({{ '/blog/starting-with-the-linearmodel-class/' | relative_url }}) was about building the container. I wrote the `LinearModel` classdef with all 26 property declarations, set up access rules so users can read properties but not accidentally overwrite them, wrote the `disp` method that prints a clean formatted summary to the screen, and added a `subsref` override that works as a gatekeeper for how the object can be accessed. At the end of that week, calling `fitlm(X, y)` would return an empty `LinearModel` object with all properties set to `[]`. The structure was correct but nothing was computed yet.

**This week** I made it real. The math engine is running, all 26 properties get populated, and `fitlm` works end to end.

## Reimplementing the fitlm Function

[fitlm](https://in.mathworks.com/help/stats/fitlm.html) is the function users call to fit a model. You pass your data and get back a fully populated [LinearModel](https://in.mathworks.com/help/stats/linearmodel.html). The function itself does not do any math. Its job is to understand what the user gave it, check for errors, and pass everything cleanly to the `LinearModel` constructor.

Here are all the valid ways to call it:

```octave
mdl = fitlm (X, y)
mdl = fitlm (tbl)
mdl = fitlm (tbl, ResponseVarName)
mdl = fitlm (tbl, y)
mdl = fitlm (..., modelspec)
mdl = fitlm (..., Name, Value, ...)
```

In simple terms, what each argument means:

- **`X`** is a numeric matrix where each column is one predictor variable (for example, house size, number of rooms, age of the house).
- **`y`** is a numeric vector containing the thing you are trying to predict (for example, house price). One value per row.
- **`tbl`** is a table where each column is a variable. When only a table is passed with no other arguments, `fitlm` treats the last column as the response and everything else as predictors.
- **`ResponseVarName`** is a string naming which column in the table is the response, when you do not want to use the last column by default.
- **`modelspec`** describes the shape of the model you want. You can pass a keyword like `'linear'` or `'quadratic'`, a Wilkinson formula string like `'Price ~ Size + Rooms'`, or a numeric terms matrix. If you skip this, `fitlm` defaults to a linear model.
- **`Name, Value`** pairs let you pass options like `'Weights'` for weighted regression, `'Exclude'` to skip certain rows, `'CategoricalVars'` to mark which columns should be treated as categories, and so on.

Inside `fitlm`, a small helper called `lm_split_args` handles the tricky part: figuring out whether the third argument in the call is a model specification or the start of a name-value pair. It does this by checking the argument against a known list of valid option names.

Commit: [reimplement fitlm function (e300446)](https://github.com/AvanishSalunke/statistics/commit/e30044680ca5437693aeabfcf6a27d8e712c4ded)

---

## LinearModel Constructor and What It Computes

Once `fitlm` has sorted out the inputs, it calls the `LinearModel` constructor in one of two forms depending on whether the user passed a matrix or a table:

```octave
LinearModel ('matrix', X, y, modelspec, Name, Value, ...)
LinearModel ('table',  tbl, ResponseVarName, modelspec, Name, Value, ...)
```

Everything from that point on happens inside the constructor. Here is a walkthrough of what it does and which internal functions it calls.

### Step 1: Parsing Options with lm_parse_nv

<p style="text-align: left;" markdown="1">The first thing the constructor does is call `lm_parse_nv`, which reads all the name-value options into a clean struct. The options it handles are `Intercept`, `Weights`, `Exclude`, `RobustOpts`, `VarNames`, `CategoricalVars`, `ResponseVar`, and `PredictorVars`. If the user passes an option name that does not exist, the function throws an error straight away rather than silently ignoring it.</p>

```octave
octave:14> fitlm (X, y, 'NotARealOption', 1)
error: LinearModel: Unknown option 'NotARealOption'.
```

### Step 2: Setting Up Variable Names

For matrix input, variable names default to `x1, x2, ..., xp, y` unless the user passed a `VarNames` option with custom names. For table input, the column names come directly from the table. The response variable name is pulled from the formula string if one was given, from the `ResponseVar` option, from a matching column name, or defaults to the last column of the table.

### Step 3: Flagging Missing and Excluded Rows

Before any math runs, the constructor scans the data for rows that should not be used in the fit. Any row with a `NaN` anywhere in the predictors or the response is flagged as missing. Any row listed in the `Exclude` option is flagged as excluded. Everything else goes into the fit. All of this is saved row by row in the `ObservationInfo` table so the user can always check exactly what happened.

### Step 4: Building the Design Matrix

Users can describe their model in two ways: a keyword like `'linear'` or `'quadratic'`, or a Wilkinson formula string like `'Price ~ Size + Rooms'`. Both need to be converted into a numeric design matrix before the solver can do anything.

For keyword or numeric terms matrix inputs, two functions handle this:

**`lm_parse_modelspec`** converts the keyword into a terms matrix. A terms matrix is a grid where each row represents one term in the model and each column represents one predictor. The value in each cell says what power of that predictor appears in that term. For two predictors `x1` and `x2`, the keywords expand like this:

- `'constant'` gives 1 term (just the intercept)
- `'linear'` gives 3 terms (intercept, x1, x2)
- `'interactions'` gives 4 terms (intercept, x1, x2, x1 times x2)
- `'purequadratic'` gives 5 terms (intercept, x1, x2, x1 squared, x2 squared)
- `'quadratic'` gives 6 terms (intercept, x1, x2, x1 times x2, x1 squared, x2 squared)

**`lm_encode_categorical`** handles columns that contain categories instead of numbers. A column with values like `'Red'`, `'Blue'`, `'Green'` cannot go into the solver directly. This function converts it into a set of binary 0/1 columns, one for each category except the first (which becomes the reference level absorbed into the intercept). So a three-level column becomes two binary columns. This is called treatment coding and it is the standard approach for categorical variables in regression.

<p style="text-align: left;" markdown="1">For Wilkinson formula strings, the constructor calls `parseWilkinsonFormula`, which is an existing function already in the statistics package. It reads the formula directly and returns the design matrix and coefficient names.</p>

### Step 5: Fitting the Model with lm_fit

`lm_fit` is the mathematical core. It takes the design matrix, the response vector, and the weights and computes everything the model needs.

The solver uses pivoted QR decomposition. The reason for this rather than the simpler classic approach is robustness. If two columns in the design matrix are nearly identical, which happens often with categorical variables, the classic approach fails. Pivoted QR handles this gracefully. It identifies which columns are redundant, solves only with the independent ones, and sets the redundant coefficients to zero with `SE = 0` and `tStat = NaN` instead of crashing.

`lm_fit` returns a struct containing: `beta` (the coefficients), `Fitted` (predicted values), `Raw` (residuals), `SSE` (sum of squared errors), `SSR` (regression sum of squares), `SST` (total sum of squares), `DFE` (degrees of freedom for error), `MSE` (mean squared error), `RMSE` (root mean squared error), `CovBeta` (coefficient covariance matrix), and `H` (the hat matrix used for diagnostics).

### Step 6: Computing Diagnostics with lm_diagnostics

Right after the fit, `lm_diagnostics` runs and computes seven observation-level statistics. These help the user understand whether any individual data point is having an unusually large influence on the model. All seven go into a `Diagnostics` table. Here is what each one means in plain terms:

**Leverage** tells you how unusual a data point's predictor values are compared to the rest of the data. A point at the edge of the predictor space has high leverage, meaning it has the potential to strongly influence the fitted line.

**Cook's Distance** gives a single number summarising how much all the fitted values across the entire dataset would change if you removed that one observation and refitted the model. A high Cook's Distance means that observation is pulling the whole model noticeably.

**Dffits** is similar to Cook's Distance but focuses on the fitted value for that specific observation. It asks: how much does the prediction for this particular point change if you leave it out of the fit?

**S2_i** is the estimated error variance of the model when observation `i` is left out. A much smaller `S2_i` compared to the full model MSE suggests that observation was inflating the error and is worth investigating.

**CovRatio** compares the precision of the coefficient estimates with and without each observation. A value close to 1 means the observation has little effect on how precisely the coefficients are estimated. Values far from 1 flag influential points.

**Dfbetas** is an `n` by `p` matrix where `n` is the number of observations and `p` is the number of coefficients. Each entry says how much one specific coefficient would change if that observation were removed. This lets you see which observations are pulling individual coefficients rather than the model as a whole.

**HatMatrix** is the full `n` by `n` matrix `H` such that `Fitted = H * y`. It captures the complete picture of how every observation influences every fitted value. Leverage is just the diagonal of this matrix, so HatMatrix is the full version of that information.

For rows that were not used in the fit (missing or excluded), Leverage is set to zero, Dfbetas and HatMatrix are filled with zeros, and everything else is set to `NaN`.

### Step 7: Assembling the Final Object

With the fit and diagnostics ready, the constructor computes the remaining statistics: `LogLikelihood`, `AIC`, `AICc`, `BIC`, `CAIC`, `Rsquared`, `Adjusted Rsquared`, and the F-statistic comparing the model to a constant-only baseline. It then builds all the output tables and assigns all 26 public properties in one go.

Commit: [LinearModel constructor, private methods and input parsing (1771c90)](https://github.com/AvanishSalunke/statistics/commit/1771c90a9172203121936db68bfed1d2a7a971fb)

---

## Verification and Usage Examples

Here is a simple example that now runs end to end:

```octave
octave:29> n = 20;
octave:30> X = [1:n; (1:n).^2]' / n;
octave:31> y = X * [3; -1] + 0.2 * sin ((1:n)');
octave:32> mdl = fitlm (X, y)
```

```
mdl =

  Linear regression model:
      y ~ 1 + x1 + x2

  Coefficients:

  3x4 table

                   Estimate        SE         tStat        pValue       
                   _________    _________    ________    ___________    

    (Intercept)     0.116189     0.112186     1.03568        0.31486    
    x1               2.50845     0.492082     5.09763    8.93779e-05    
    x2             -0.978835    0.0227611    -43.0048              0    


Number of observations: 20, Error degrees of freedom: 17
Root Mean Squared Error: 0.150791
R-squared: 0.999338,  Adjusted R-Squared: 0.99926
F-statistic vs. constant model: 12831.5, p-value = 0
```

You can also query individual properties directly:

```octave
mdl.NumObservations            ## 20
mdl.DFE                        ## 17
mdl.Rsquared.Ordinary          ## 0.999338
mdl.Diagnostics.Leverage       ## 20x1 vector, one value per row
mdl.Diagnostics.CooksDistance  ## 20x1 vector
```

Missing value handling works automatically. If one row has a `NaN`, it gets excluded and the model fits on the rest:

```octave
X2 = X;
X2(5, 1) = NaN;
m2 = fitlm (X2, y);

m2.NumObservations              ## 19, row 5 was dropped
m2.Fitted(5)                    ## NaN
m2.ObservationInfo.Missing(5)   ## true
m2.SSE + m2.SSR                 ## equals m2.SST exactly
```

---

## Mentor Feedback

Before this project, Octave already had an older version of `fitlm`. That older version returned a cell array `TAB` and a struct `STATS`, and its test suite checked specific numeric values:

```octave
[TAB, STATS] = fitlm (X, y, 'linear', 'CategoricalVars', 1);
assert (TAB{2,2}, 10, 1e-04);
assert (TAB{3,2}, 7.99999999999999, 1e-09);
assert (TAB{2,6}, 9.82555903510687, 1e-09);
```

<p style="text-align: left;" markdown="1">When I rewrote `fitlm` to return a `LinearModel` object, I deleted those tests because the output format had completely changed. My mentors [Colin B. MacDonald](https://octave.discourse.group/u/cbm/summary) and [Andreas Bertsatos](https://octave.discourse.group/u/pr0m1th3as/summary) pointed out that we should preserve these tests by refactoring them to work with the new class structure. Since the underlying mathematics remains identical, the computed coefficient estimates, standard errors, and t-statistics should produce the same numeric values regardless of what container they come back in. The right approach is to update the tests to access the same values through the new object properties. The old test `assert(TAB{2,2}, 10, 1e-04)` becomes `assert(mdl.Coefficients.Estimate(1), 10, 1e-04)`. Same number, different path.</p>

This was a useful suggestion. Tests that only check `class(fitlm(X, y)) == 'LinearModel'` pass even when all the math is wrong. They just confirm the function ran without crashing. Checking actual numeric values is what makes a test suite meaningful.

Andreas also pointed out that the `lm_parse_nv` function I wrote inside `LinearModel` does essentially the same thing as an existing utility in the `datatypes` package. The recommendation was to call that existing function instead of keeping a separate copy. Beyond saving a few lines, it keeps behaviour consistent across the whole package, which matters in an open-source project where many people work on the same codebase. Andreas also mentioned timing: there is a second GSoC contributor this summer working on `anova`, and their work will depend on `fitlm` being ready. That adds some urgency to getting everything reviewed and merged.

## Development Branch

All the code is on this branch:
[gsoc-2026-linearmodel](https://github.com/AvanishSalunke/statistics/tree/gsoc-2026-linearmodel)

## Next Steps

Before moving forward, I will first work on incorporating the changes suggested by my mentors. After that, the next things to build are the methods that sit on top of the fitted model: `predict` for making predictions on new data, `coefCI` for confidence intervals on the coefficients, `coefTest` for statistical tests, and other methods. The groundwork is already done since the constructor stores everything these methods need.

Stay tuned for the next update!