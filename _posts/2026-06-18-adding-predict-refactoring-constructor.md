---
layout: post
title: "Building on top of the fitted model"
date: 2026-06-18
description: "This week started with fixing the things my mentors pointed out, then moved into adding the first real user-facing method, predict, along with a few internal refactors that will make the upcoming stepwiselm much easier to build."
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

<p class="intro"><span class="dropcap">L</span>ast week the <a href="https://in.mathworks.com/help/stats/linearmodel.html">LinearModel</a> constructor became fully functional and <a href="https://in.mathworks.com/help/stats/fitlm.html">fitlm</a> worked end to end for the first time. This week was about making what we had cleaner before adding new things. I applied the changes suggested by my mentors <a href="https://octave.discourse.group/u/cbm/summary">Colin B. MacDonald</a> and <a href="https://octave.discourse.group/u/pr0m1th3as/summary">Andreas Bertsatos</a>, refactored parts of the constructor, and then implemented the first user-facing method: <code>predict</code>.</p>

## Project Status

[**Week 1**]({{ '/blog/introduction-and-gsoc-timeline/' | relative_url }}) was project planning and setup.

[**Week 2**]({{ '/blog/starting-with-the-linearmodel-class/' | relative_url }}) was the class shell: properties, access rules, the display method, and the `subsref` gatekeeper.

[**Week 3**]({{ '/blog/improving-fitlm-function-and-expanding-linearmodel-constructor/' | relative_url }}) was the constructor and `fitlm`: the math solver, all the diagnostics, the full 26-property object, and the front-end parser.

**This week** was fixing what the mentors flagged, refactoring the constructor to be cleaner, and adding the first methods that users can actually call on the object.

---

## Applying Mentor Feedback First

Before writing any new code, I went back and fixed the two things the mentors pointed out last week. I wanted to get those right before building on top of them.

### Using What Already Exists in the Package

The `lm_parse_nv` function I had written inside `LinearModel` was essentially a custom version of something that already exists in the datatypes package. It read through name-value argument pairs one by one, matched them against known option names, and threw an error if an unknown name was passed.

Andreas pointed out that the `datatypes` package has a utility function called `parsePairedArguments` that does this job in a standardised way. The recommendation was to call that instead of keeping a separate copy of the same logic inside `LinearModel`.

The change was straightforward. Instead of a manual loop, `lm_parse_nv` now just declares the option names and their default values, calls `parsePairedArguments`, and checks if anything unrecognised was left over. It is shorter, consistent with the rest of the package, and handles edge cases the same way every other function in the codebase does.

### Fixing the BISTs to Actually Test the Math

The second thing I fixed was the test suite. When I rewrote `fitlm` to return a `LinearModel` object, I had removed the old tests because the output format had completely changed. My mentors pointed out that the underlying numbers should still be the same, so instead of deleting the old tests, I should update them to access the same values through the new object.

The old `fitlm` returned a cell array `TAB` where each row was a coefficient and the columns held the estimate, standard error, lower CI, upper CI, t-statistic, and p-value. So `TAB{2,2}` was the estimate of the first coefficient, `TAB{2,3}` was its standard error, and `TAB{2,6}` was its t-statistic. Those same values now live inside the `Coefficients` table of the `LinearModel` object. The mapping looks like this:

```octave
## Old way                          New way
TAB{2,2}  ## estimate (intercept)   mdl.Coefficients.Estimate(1)
TAB{3,2}  ## estimate (x1)          mdl.Coefficients.Estimate(2)
TAB{2,3}  ## SE (intercept)         mdl.Coefficients.SE(1)
TAB{2,6}  ## tStat (intercept)      mdl.Coefficients.tStat(1)
TAB{2,7}  ## pValue (intercept)     mdl.Coefficients.pValue(1)
```

The actual numbers being checked are identical. For example, the intercept estimate in the categorical variable test case was 10, and the t-statistic for it was 9.82555903510687. Those same values now need to come out of `mdl.Coefficients.Estimate(1)` and `mdl.Coefficients.tStat(1)`. The math has not changed, only the path to reach the result has.

## Refactoring the Constructor

With the mentor changes done, I also cleaned up a few things in the constructor that were going to cause problems later.

### Simplifying How fitlm Calls the Constructor

In last week's version, `fitlm` passed a string as the first argument to the `LinearModel` constructor to signal whether the data was a matrix or a table. The constructor then used that string to decide which code path to take.

This was unnecessary. The constructor can just check directly whether the first argument is a table using `istable()`. So I removed the string flag. `fitlm` now passes the data directly, and the constructor figures out the type itself. One less thing to pass around and one less thing that could be mismatched.

### Extracting lm_build_design

In the original constructor, there was an inline loop that built the design matrix by going through each row of the terms matrix and assembling the corresponding column. That loop needed to be shared with the new `predict` method, since prediction also requires building a design matrix from the same terms.

Rather than copy the loop in two places, I pulled it out into a private static method called `lm_build_design`. It takes the terms matrix and the encoded predictor matrix as inputs and returns the design matrix. Both the constructor and `predict` now call it the same way, so the logic lives in exactly one place.

### Extracting lm_criteria

The constructor had a large block of math near the end that computed LogLikelihood, AIC, AICc, BIC, CAIC, R-squared, Adjusted R-squared, and the F-statistic. These are all model quality measures that tell you how good the fit is.

I pulled all of that into a separate private method called `lm_criteria`. It takes the fit struct, the number of observations, and whether the model has an intercept, and returns all those values in a struct.

The reason this matters beyond just keeping the constructor tidy is that `stepwiselm`, which comes later, needs to evaluate these same criteria for every candidate model it tries during the selection process. Instead of building a full `LinearModel` object for each candidate (which would be slow), the stepwise loop can call `lm_fit` and then `lm_criteria` directly to get just the numbers it needs to decide which model to pick. The full object only gets built once at the very end for the winning model.

---

## New Public Methods

With those cleanups done, I added the first methods that users can call on a fitted model.

### predict

[predict](https://www.mathworks.com/help/stats/linearmodel.predict.html) is the method you use to apply a fitted model to new data. You hand it a matrix or a table of new predictor values and it gives back the predicted response values for those inputs.

```octave
ypred = predict (mdl, Xnew)
ypred = predict (mdl)
[ypred, yci] = predict (mdl, Xnew)
[ypred, yci] = predict (mdl, Xnew, Name, Value)
```

When called with no new data, it returns the predicted values for all the training observations. When called with `Xnew`, it re-encodes any categorical columns using the same mapping that was stored during the original fit, builds the design matrix from the stored `TermsMatrix`, and computes the predictions.

The optional second output `yci` is a matrix with two columns giving the lower and upper confidence interval bounds for each prediction. There are a few Name-Value options that control what kind of interval you get:

**`'Alpha'`** sets the significance level. The default is 0.05, which gives 95% intervals.

**`'Prediction'`** lets you choose between two types of interval. `'curve'` (the default) gives a confidence interval on the mean response at that point. Think of it as the range where the true average outcome is likely to fall. `'observation'` gives a prediction interval for a single new observation. This is always wider because it also accounts for the natural scatter around the mean, not just uncertainty in the estimated line.

**`'Simultaneous'`** switches to simultaneous bounds using Scheffe's method. These cover the entire fitted curve at the stated confidence level, which matters when you are making predictions over a range of inputs rather than at a single point.

Here is what it looks like in practice:

```octave
octave:25> load carsmall
octave:26> X = [Weight, Horsepower];
octave:27> mdl = fitlm (X, MPG, 'linear');
octave:28> Xnew = [3000, 120; 4000, 200];
octave:29> [ypred, yci] = predict (mdl, Xnew)
```

```text
ypred =

   23.032
   13.105

yci =

   22.131   23.932
   11.082   15.128
```

The first row is for a 3000 lb car with 120 hp, predicted to get about 23 MPG. The second is a heavier, more powerful car at about 13 MPG. The intervals show the uncertainty around each prediction. Points that are closer to the centre of the training data get narrower intervals.

You can also see the difference between the mean confidence interval and the single-observation prediction interval:

```octave
[~, yci_mean] = predict (mdl, Xnew, 'Prediction', 'curve');
[~, yci_obs]  = predict (mdl, Xnew, 'Prediction', 'observation');
```

`yci_obs` will always be wider than `yci_mean` because it accounts for individual variation on top of the model uncertainty.

### random

`random` does the same thing as `predict` but adds random noise on top of each predicted value. The noise is drawn from a normal distribution with mean zero and standard deviation equal to the model's `RMSE`. This simulates what a new real-world observation would look like if you actually went and collected it.

```octave
ysim = random (mdl, Xnew)
```

Calling it twice on the same input will give slightly different results each time because of the randomness. This is useful for simulation studies and for checking how much variability to expect in new data.

## New Private Helpers

Along with the two public methods, two more private helpers were added.

**`lm_predict`** handles the re-encoding step for categorical variables when `predict` receives new data. When a model was trained on a column with categories like `'air'` and `'oil'`, a new row with `'oil'` needs to be converted using the exact same integer mapping that was used during training. `lm_predict` does this lookup by reading the `CatLevelInfo` struct that the constructor stored during the original fit.

**`lm_refit`** is a helper that will be used by `stepwiselm`, `addTerms`, and `removeTerms`. It takes the current model object and a new terms matrix, reads back the original fit options from a stored `OrigOpts` property, and calls `fitlm` again to produce a new model with a different structure. This means whether you are letting `stepwiselm` search for the best model automatically or manually adding and removing terms yourself, the same underlying refit logic handles it without you needing to re-specify all your original settings each time.

To support `lm_refit`, two new hidden properties were also added to the class: `EncodedPredMatrix`, which caches the encoded predictor matrix so it does not have to be recomputed on every refit, and `OrigOpts`, which stores the parsed Name-Value options from the original `fitlm` call.

---

## Development Branch

All the code is on this branch:
[gsoc-2026-linearmodel](https://github.com/AvanishSalunke/statistics/tree/gsoc-2026-linearmodel)

--- 

## Next Steps

With `predict` and `random` working, the next methods to implement are:

- [coefCI](https://www.mathworks.com/help/stats/linearmodel.coefci.html) returns confidence intervals for each coefficient, showing the range of values each estimate could reasonably take.
- [coefTest](https://www.mathworks.com/help/stats/linearmodel.coeftest.html) runs a hypothesis test on any linear combination of coefficients, useful for checking whether a group of predictors actually matters.
- [feval](https://in.mathworks.com/help/stats/linearmodel.feval.html) evaluates the model on new data by going through the design matrix path directly, similar to predict but at a lower level.
- [dwtest](https://in.mathworks.com/help/stats/linearmodel.dwtest.html) runs the Durbin-Watson test to check whether the residuals have autocorrelation, which would mean the model is missing a pattern in the data.
- [addTerms](https://www.mathworks.com/help/stats/linearmodel.addterms.html) adds new predictor terms to an already fitted model without having to refit from scratch.
- [removeTerms](https://in.mathworks.com/help/stats/linearmodel.removeterms.html) removes terms from a fitted model, again without starting over.

Stay tuned for the next update!
