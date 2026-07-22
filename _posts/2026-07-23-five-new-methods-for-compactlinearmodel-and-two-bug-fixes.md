---
layout: post
title: "Five new methods for CompactLinearModel and two bug fixes"
date: 2026-07-23
description: "This week brought predict, feval, random, coefCI, and coefTest to CompactLinearModel, along with a coefficient ordering fix and a feval bug fix."
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

<p class="intro"><span class="dropcap">L</span>ast week the initial <a href="https://in.mathworks.com/help/stats/classreg.regr.compactlinearmodel.html">CompactLinearModel</a> classdef landed with just its properties and constructor. This week the class actually became usable: <code>coefCI</code>, <code>coefTest</code>, <code>predict</code>, <code>random</code>, and <code>feval</code> were all added and checked against MATLAB. Along the way two real bugs turned up and got fixed - one where coefficients could come out in the wrong order compared to MATLAB, and one where <code>feval</code> didn't handle a categorical value passed in as plain text.</p>

## Project Status

<div style="text-align: left; margin: 1rem 0;">
  <p style="margin-bottom: 0.8rem; text-align: left;"><a href="{{ '/blog/introduction-and-gsoc-timeline/' | relative_url }}"><strong>Week 1</strong></a> was project planning and setup.</p>
  <p style="margin-bottom: 0.8rem; text-align: left;"><a href="{{ '/blog/starting-with-the-linearmodel-class/' | relative_url }}"><strong>Week 2</strong></a> was the class shell: properties, access rules, the display method, and the <code>subsref</code> gatekeeper.</p>
  <p style="margin-bottom: 0.8rem; text-align: left;"><a href="{{ '/blog/improving-fitlm-function-and-expanding-linearmodel-constructor/' | relative_url }}"><strong>Week 3</strong></a> was the constructor and <code>fitlm</code>: the math solver, all the diagnostics, the full 26-property object, and the front-end parser.</p>
  <p style="margin-bottom: 0.8rem; text-align: left;"><a href="{{ '/blog/adding-predict-refactoring-constructor/' | relative_url }}"><strong>Week 4</strong></a> was applying mentor feedback, refactoring the constructor, and adding <code>predict</code> and <code>random</code>.</p>
  <p style="margin-bottom: 0.8rem; text-align: left;"><a href="{{ '/blog/statistical-tests-autocorrelation-checks-and-model-modification/' | relative_url }}"><strong>Week 5</strong></a> was <code>feval</code>, <code>coefCI</code>, <code>coefTest</code>, <code>dwtest</code>, and <code>addTerms</code>.</p>
  <p style="margin-bottom: 0.8rem; text-align: left;"><a href="{{ '/blog/removeterms-three-new-plotting-methods-and-bug-fixes/' | relative_url }}"><strong>Week 6</strong></a> was <code>removeTerms</code>, <code>plotResiduals</code>, <code>plotDiagnostics</code>, <code>plotEffects</code>, and a set of bug fixes.</p>
  <p style="margin-bottom: 0.8rem; text-align: left;"><a href="{{ '/blog/midterm-evaluation-two-new-plotting-methods-and-robust-regression/' | relative_url }}"><strong>Week 7</strong></a> was the midterm evaluation, robust regression, <code>plotAdjustedResponse</code>, and <code>plotAdded</code>.</p>
  <p style="margin-bottom: 0.8rem; text-align: left;"><a href="{{ '/blog/two-new-plotting-methods-the-compact-method-and-compactlinearmodel/' | relative_url }}"><strong>Week 8</strong></a> was <code>plot</code>, <code>plotInteraction</code>, <code>compact</code>, and the initial <code>CompactLinearModel</code> structure.</p>
  <p style="margin-bottom: 0.8rem; text-align: left;"><strong>This week</strong> was <code>coefCI</code>, <code>coefTest</code>, <code>predict</code>, <code>random</code>, and <code>feval</code> on <code>CompactLinearModel</code>, plus two bug fixes.</p>
</div>

---

## coefCI on CompactLinearModel - [here](https://github.com/AvanishSalunke/statistics/commit/7579b78c1913ab1777b8e3497d0d23c2cb79a133)

[coefCI](https://in.mathworks.com/help/stats/classreg.regr.compactlinearmodel.coefci.html) returns a confidence interval for every coefficient in the model. Each row of the output is one coefficient's lower and upper bound, at whatever significance level you ask for (95% by default).

```octave
octave:8> cmdl = compact (mdl);
octave:9> ci = coefCI (cmdl)
```

```text
ci =

  -0.1205   0.3529
   1.4702   3.5467
  -1.0269  -0.9308
```

```octave
ci = coefCI (cmdl, 0.01)   ## 99% intervals instead of the default 95%
```

A narrower confidence level gives a tighter interval, and a wider one gives a looser interval, since the method just scales the width of each bound based on the coefficient's standard error.

## coefTest on CompactLinearModel - [here](https://github.com/AvanishSalunke/statistics/commit/e0da329a7926f4f3cb2ed5495c50551fc6c4ae60)

[coefTest](https://in.mathworks.com/help/stats/classreg.regr.compactlinearmodel.coeftest.html) checks whether one or more coefficients are equal to some value, usually zero. Called with no arguments it runs the overall model test - are all the coefficients except the intercept zero?

```octave
octave:22> [p, F, r] = coefTest (cmdl)
```

```text
p = 9.4899e-28
F = 1.2831e+04
r = 2
```

You can also give it your own hypothesis as a contrast matrix, to test something more specific, like whether two coefficients are equal to each other:

```octave
octave:31> H = [0 1 -1];
octave:32> [p, F, r] = coefTest (cmdl, H)
```

```text
p = 3.2021e-06
F = 45.992
r = 1
```

A small p-value means the data gives strong evidence against the hypothesis being tested.

## predict on CompactLinearModel - [here](https://github.com/AvanishSalunke/statistics/commit/3dd948a67c06bcb1e35eec754b1f9f7ae23739ae)

[predict](https://in.mathworks.com/help/stats/classreg.regr.compactlinearmodel.predict.html) computes the fitted response at new predictor values, with optional confidence or prediction intervals.

```octave
octave:33> Xnew = [0.5 0.25; 1.0 1.0];
octave:34> [ypred, yci] = predict (cmdl, Xnew)
```

```text
ypred =

   1.1257
   1.6458

yci =

   0.8102   1.4412
   0.8582   2.4334
```

Options let you change the confidence level, ask for a simultaneous bound across all points instead of a pointwise one, and switch between a confidence interval on the mean response or a wider prediction interval for a single future observation.

```octave
[ypred, yci] = predict (cmdl, Xnew, 'Alpha', 0.01)
[ypred, yci] = predict (cmdl, Xnew, 'Simultaneous', true)
[ypred, yci] = predict (cmdl, Xnew, 'Prediction', 'observation')
```

## random on CompactLinearModel - [here](https://github.com/AvanishSalunke/statistics/commit/2623a991a8e918581b5cb6d8ac0e5d6d89249ff6)

[random](https://in.mathworks.com/help/stats/classreg.regr.compactlinearmodel.random.html) simulates new response values. It computes the fitted response the same way `predict` does, and then adds a bit of random noise on top, matching the amount of noise estimated in the original fit.

```octave
octave:37> ysim = random (cmdl, Xnew)
```

```text
ysim =

   1.1452
   1.9922
```

```octave
octave:38> ysim = random (cmdl, Xnew)
```

```text
ysim =

   1.1999
   1.8687
```

Because the noise is random, calling `random` twice with the same input gives two different answers each time.

## feval on CompactLinearModel - [here](https://github.com/AvanishSalunke/statistics/commit/0c74f195860fe6b6f78d822c3a0006912ae29c87)

[feval](https://in.mathworks.com/help/stats/classreg.regr.compactlinearmodel.feval.html) also returns fitted predictions, but instead of taking one matrix of predictor values, it takes each predictor as its own separate argument. This makes a fitted model usable anywhere a plain function is expected.

```octave
feval (cmdl, x1, x2)     ## one value per predictor
feval (cmdl, X)          ## a full matrix in one go, same as predict
```

## Bug Fix: passing a category as plain text to feval - [here](https://github.com/AvanishSalunke/statistics/commit/b9d4cba0f3fa6817d5b016b181302c0889fb6e52)

`feval` supports passing predictor values as separate arguments instead of one matrix. For a categorical predictor, one of those arguments can be the category name itself as plain text, like a year written as `'70'`, rather than its numeric code.

This case wasn't handled correctly. A short piece of text wasn't treated as a single value the way a plain number is, so it got confused with mismatched-size numeric data and rejected. A single-character category would have been silently swapped for its character code instead of its real category, which is just as wrong even though nothing had caught it.

```octave
feval (cmdl, w, '70')             ## text category, resolved to its numeric code
feval (cmdl, w, categorical (70)) ## a categorical value works the same way
feval (cmdl, w, '99')             ## a category that doesn't exist gives NaN
```

The fix checks whether an argument is text or a category first, looks up which numeric code it corresponds to, and only then continues with the normal calculation.

## Bug Fix: coefficients coming out in the wrong order - [here](https://github.com/AvanishSalunke/statistics/commit/4ee114294aebffebee68a9762582df7ae2514c3c)

The order coefficients appear in a fitted model should follow the order of the columns in the input data table. Internally, this order was instead being sorted alphabetically by variable name, so a model with predictors named, say, `Weight` and `Year` could end up listing `Year` first even though `Weight` was the first column in the table.

```text
Expected : (Intercept), Horsepower, Weight, Origin_France, ...
Got      : (Intercept), Horsepower, Origin_France, ..., Weight
```

The fix reorders the internal variable list, and the coefficients built from it, to follow the data table's actual column order rather than an alphabetical sort.

## Bug Fix: model display said "Coefficients:" instead of "Estimated Coefficients:" - [here](https://github.com/AvanishSalunke/statistics/commit/5e16bb914f3281ea135571b109ee8113a919370a)

The header printed above the coefficient table when displaying a fitted model said `Coefficients:`. It should say `Estimated Coefficients:`. A small text change fixes the display for both the full model and the compact model.


---

## Development Branch

All the code is on this branch:
[gsoc-2026-linearmodel](https://github.com/AvanishSalunke/statistics/tree/gsoc-2026-linearmodel)

---

## Next Steps

- `plotEffects` and `plotInteraction` on `CompactLinearModel` - main-effect and interaction-effect plots for a fitted model.
- [stepwiselm](https://in.mathworks.com/help/stats/stepwiselm.html) - automated model selection by forward and backward stepping.

Stay tuned for the next update!
