---
layout: post
title: "Five new methods for LinearModel"
date: 2026-06-25
description: "This week the LinearModel gained five new methods: feval, coefCI, coefTest, dwtest, and addTerms."
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

<p class="intro"><span class="dropcap">B</span>y the end of last week, the <a href="https://in.mathworks.com/help/stats/linearmodel.html">LinearModel</a> could fit a model, make predictions, and simulate new observations. The public interface had only two methods though: <code>predict</code> and <code>random</code>. This week five more were added: <code>feval</code>, <code>coefCI</code>, <code>coefTest</code>, <code>dwtest</code>, and <code>addTerms</code>. Before any of that was written, a mentor meeting changed how the internal helper functions are organised, cleared up a question about overloading a built-in Octave function, and corrected the handling of the <code>'full'</code> model specification.</p>

## Project Status

[**Week 1**]({{ '/blog/introduction-and-gsoc-timeline/' | relative_url }}) was project planning and setup.

[**Week 2**]({{ '/blog/starting-with-the-linearmodel-class/' | relative_url }}) was the class shell: properties, access rules, the display method, and the `subsref` gatekeeper.

[**Week 3**]({{ '/blog/improving-fitlm-function-and-expanding-linearmodel-constructor/' | relative_url }}) was the constructor and `fitlm`: the math solver, all the diagnostics, the full 26-property object, and the front-end parser.

[**Week 4**]({{ '/blog/building-on-top-of-the-fitted-model/' | relative_url }}) was applying mentor feedback, refactoring, and adding `predict` and `random`.

**This week** was a couple of bug fixes and five new public methods.

## Meeting with my mentor

I had a meeting with my mentor [Andreas Bertsatos](https://octave.discourse.group/u/pr0m1th3as/summary) before writing any new code. Three important things came out of it.

### Supporting the `'full'` Model Specification

Last week's code threw an error when a user passed `'full'` as the modelspec. The error was not in `fitlm` itself - `fitlm` just takes whatever modelspec you give it and passes it straight through to the `LinearModel` constructor. The error was inside `lm_parse_modelspec`, which is the function that actually interprets the modelspec string. It had an explicit line that said `'full'` is invalid and stopped there.

My mentor pointed out that `'full'` is actually valid and should be supported. It means the full factorial model - every possible combination of the predictors as interaction terms, not just pairwise ones. For two predictors it gives the same result as `'interactions'`. The difference shows up with three or more predictors, where `'full'` also adds higher-order interactions like `x1:x2:x3` that `'interactions'` leaves out. This is particularly useful in designed experiments where researchers want to model every possible way the factors can combine and interact with each other.

The old test that checked for the error was removed and replaced with a passing test confirming the call now works correctly.

```octave
octave:8> mdl = fitlm (X, y, 'full')
```

```text
mdl =

  Linear regression model:
      y ~ 1 + x1 + x2 + x1:x2

  Coefficients:

  4x4 table

                   Estimate        SE         tStat        pValue       
                   _________    _________    ________    ___________    

    (Intercept)     0.157641     0.169192    0.931725       0.365326    
    x1               2.08543      1.36153     1.53167       0.145135    
    x2             -0.929683     0.148744    -6.25021    1.15922e-05    
    x1:x2          -0.031208    0.0932669    -0.33461       0.742266    


Number of observations: 20, Error degrees of freedom: 16
Root Mean Squared Error: 0.154891
R-squared: 0.999343,  Adjusted R-Squared: 0.999219
F-statistic vs. constant model: 8107.51, p-value = 1.1642e-25
```

### Reorganising Where the Helper Functions Live

My mentor pointed out that helper functions which do not need to be documented or called from outside the class can be made private by placing them after `endclassdef` in the same file. Functions written there are not accessible to users and do not appear in the documentation.

The functions that moved out are `lm_parse_nv`, `lm_parse_modelspec`, `lm_encode_categorical`, `lm_diagnostics`, and `lm_refit`. The functions that stayed as private static methods inside the class are `lm_fit`, `lm_build_design`, `lm_criteria`, and `lm_predict`.

### The `feval` Question

Before implementing `feval`, I was not sure whether it was safe to name a class method the same as the built-in Octave function `feval`. I was worried that adding a method with that name would somehow clash with the global one.

My mentor explained how Octave's dispatcher actually works. When Octave sees a function call, it first checks the type of the first argument. If that argument is a classdef object and the class defines a method with that same name, Octave calls the class method, not the built-in. The global built-in is only reached when no matching class method exists. So calling `feval(mdl, x)` where `mdl` is a `LinearModel` will always go to the `LinearModel.feval` method. Calling `feval('sin', x)` where the first argument is a string still goes to the global Octave built-in as normal. There is no clash because Octave routes the call based on the type of the first argument.

This is the same mechanism that makes `size`, `disp`, and `display` work differently depending on what you pass them. Classes can overload any function name and Octave handles the routing transparently.

## Bug Fixes Before the New Methods

### F-statistic p-value in lm_criteria

The F-statistic p-value was computed using `1 - fcdf(Fstat, df1, DFE)`. For very large F values, which happen when the model fits the data extremely well, `fcdf` returns exactly `1.0` because the probability is so close to 1 that double-precision floating point cannot represent the difference. Subtracting 1 from 1 then gives exactly zero, so the p-value would come out as 0 instead of the correct tiny value like `1.16e-25`.

The fix replaces this with `betainc`, which computes the upper-tail probability directly without needing to subtract anything. It stays accurate no matter how large the F value gets. A small guard was also added to catch degenerate models, for example a saturated model where the number of parameters equals the number of observations, where the F-test is not meaningful and the result should be `NaN` rather than an error.

### Intercept flag in lm_refit

`lm_refit` is the function called whenever you modify a model by adding or removing terms. The old version always used the original `Intercept` setting from when the model was first fitted. This caused a subtle problem: if you fitted a model without an intercept and then called `addTerms(mni, '1')` to add one back in, `lm_refit` would see the original `Intercept = false` and quietly strip the intercept row right back out during the refit. The fix was to look at the new combined terms matrix itself and check whether an intercept row is present, rather than relying on the stored original setting.

## New Methods

### feval

[feval](https://in.mathworks.com/help/stats/linearmodel.feval.html) evaluates a fitted model at new predictor values. It works just like `predict` but gives you two ways to pass the inputs - either as one matrix with all predictors, or as separate arguments one per predictor.

```octave
## matrix input - one column per predictor
ypred = feval (mdl, [0.5, 0.25; 1.0, 1.0])

## separate arguments - one per predictor
ypred = feval (mdl, 0.5, 0.25)

## mixing scalars and vectors - scalar is broadcast to match the vector length
ypred = feval (mdl, 0.5, [0.1; 0.2; 0.3])
```

The separate-argument form matches how MATLAB's `feval` works on model objects, so implementing both forms keeps the behaviour consistent with MATLAB.

### coefCI

[coefCI](https://in.mathworks.com/help/stats/linearmodel.coefci.html) returns confidence intervals for each coefficient estimate. A confidence interval gives you a range of values the true coefficient is likely to fall within. By default it gives 95% intervals, but you can choose any level.

The output is a matrix with one row per coefficient. The first column is the lower bound and the second is the upper bound.

```octave
octave:9> ci = coefCI (mdl)
```

```text
ci =

  -0.2010   0.5163
  -0.8009   4.9718
  -1.2450  -0.6144
  -0.2289   0.1665
```

```octave
octave:10> ci = coefCI (mdl, 0.01)
```

```text
ci =

  -0.3365   0.6518
  -1.8913   6.0622
  -1.3641  -0.4952
  -0.3036   0.2412
```

The interval for the intercept includes zero, so we cannot say it is significantly different from zero. The interval for `x2` is entirely negative, confirming it has a real negative effect on the response. A narrow interval means a precise estimate; a wide one means high uncertainty.

### coefTest

[coefTest](https://in.mathworks.com/help/stats/linearmodel.coeftest.html) runs an F-test on the model's coefficients. The default call tests whether all the non-intercept coefficients are jointly zero, which is the same question as the overall F-test in the model summary.

```octave
## overall test
p = coefTest (mdl)

## get p-value, F-statistic, and numerator degrees of freedom
[p, F, r] = coefTest (mdl)

## test a specific coefficient - H is a row vector, one entry per coefficient
## this tests whether the x1 coefficient is zero
p = coefTest (mdl, [0, 1, 0])

## test a joint hypothesis - each row of H is one combination to test
## this tests whether x1 and x2 are both zero at the same time
p = coefTest (mdl, [0 1 0; 0 0 1])

## test H*beta = C instead of H*beta = 0
p = coefTest (mdl, [0, 1, 0], 2)
```

### dwtest

[dwtest](https://in.mathworks.com/help/stats/linearmodel.dwtest.html) checks whether the residuals of the model are autocorrelated. This matters when your data has a natural order, like time-series measurements, because autocorrelation means the model is missing some pattern in the data and the standard errors it reports cannot be trusted.

<p style="text-align: left;" markdown="1">The Durbin-Watson statistic always falls between 0 and 4. Near 2 means no autocorrelation. Near 0 means positive autocorrelation (consecutive residuals tend to have the same sign). Near 4 means negative autocorrelation (consecutive residuals alternate in sign).</p>

```octave
## default call - exact method, two-sided test
[p, DW] = dwtest (mdl)

## choose how the p-value is computed: 'exact' or 'approximate'
[p, DW] = dwtest (mdl, 'approximate')

## choose which direction to test: 'both', 'right' (positive AC), 'left' (negative AC)
[p, DW] = dwtest (mdl, 'exact', 'right')
```

### addTerms

[addTerms](https://in.mathworks.com/help/stats/linearmodel.addterms.html) adds new terms to an already fitted model and returns a fresh fitted model with those terms included. The original model is never changed.

```octave
## string input using Wilkinson notation
NewMdl = addTerms (mdl, 'x1:x2')          ## interaction term
NewMdl = addTerms (mdl, 'x2^2')           ## quadratic term
NewMdl = addTerms (mdl, 'x1*x2')          ## crossing: adds x1, x2, and x1:x2
NewMdl = addTerms (mdl, 'x1^2 + x1:x2')  ## two terms at once
NewMdl = addTerms (mni, '1')              ## add intercept to a no-intercept model

## numeric matrix input - each row is one term, columns are predictor exponents
NewMdl = addTerms (mdl, [1, 1, 0])  ## same as 'x1:x2'
NewMdl = addTerms (mdl, [1, 1])     ## auto-padded to [1, 1, 0], same result
```

Terms already in the model are silently skipped. If everything you asked to add is already there, a warning is issued and the original model is returned.

All the original settings - weights, excluded rows, categorical variable flags - are preserved automatically in the new model. So if you originally used `'Weights'`, the new model uses the same weights without you needing to pass them again.

```octave
mw     = fitlm (X, y, 'Weights', w);
mw_new = addTerms (mw, 'x1:x2');   ## weights carried over automatically
```

## Development Branch

All the code is on this branch:
[gsoc-2026-linearmodel](https://github.com/AvanishSalunke/statistics/tree/gsoc-2026-linearmodel)

## Next Steps

With `addTerms` working, the next method to implement is [removeTerms](https://in.mathworks.com/help/stats/linearmodel.removeterms.html). It works the same way as `addTerms` but in reverse - you pass the terms you want to drop and it returns a new model without them.

After that, the focus shifts to the plotting methods:

- [plot](https://in.mathworks.com/help/stats/linearmodel.plot.html) - scatter plot of the fitted values against the observed response
- [plotResiduals](https://in.mathworks.com/help/stats/linearmodel.plotresiduals.html) - shows the residuals in various formats to help spot pattern or assumption violations
- [plotDiagnostics](https://in.mathworks.com/help/stats/linearmodel.plotdiagnostics.html) - visualises influence statistics like Cook's Distance and leverage to identify unusual observations
- [plotAdded](https://in.mathworks.com/help/stats/linearmodel.plotadded.html) - shows the effect of adding one predictor after accounting for all the others already in the model
- [plotAdjustedResponse](https://in.mathworks.com/help/stats/linearmodel.plotadjustedresponse.html) - shows the relationship between one predictor and the response after removing the effect of all other predictors
- [plotEffects](https://in.mathworks.com/help/stats/linearmodel.ploteffects.html) - shows how the predicted response changes as each predictor varies
- [plotInteraction](https://in.mathworks.com/help/stats/linearmodel.plotinteraction.html) - shows how the effect of one predictor changes at different levels of another

Stay tuned for the next update!
