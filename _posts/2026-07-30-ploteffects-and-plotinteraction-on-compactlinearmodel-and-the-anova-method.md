---
layout: post
title: "plotEffects and plotInteraction on CompactLinearModel, and the anova method"
date: 2026-07-30
description: "This week plotEffects and plotInteraction were extended to CompactLinearModel using a new cached-contrasts approach, and the anova method was added to LinearModel."
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

<p class="intro"><span class="dropcap">L</span>ast week five methods landed on <a href="https://in.mathworks.com/help/stats/classreg.regr.compactlinearmodel.html">CompactLinearModel</a>. This week added two more - <code>plotEffects</code> and <code>plotInteraction</code> - and brought the <a href="https://in.mathworks.com/help/stats/linearmodel.anova.html">anova</a> method to <code>LinearModel</code> for the first time.</p>

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
  <p style="margin-bottom: 0.8rem; text-align: left;"><a href="{{ '/blog/five-new-methods-for-compactlinearmodel-and-two-bug-fixes/' | relative_url }}"><strong>Week 9</strong></a> was <code>coefCI</code>, <code>coefTest</code>, <code>predict</code>, <code>random</code>, and <code>feval</code> on <code>CompactLinearModel</code>, plus two bug fixes.</p>
  <p style="margin-bottom: 0.8rem; text-align: left;"><strong>This week</strong> was <code>plotEffects</code> and <code>plotInteraction</code> on <code>CompactLinearModel</code>, and the <code>anova</code> method on <code>LinearModel</code>.</p>
</div>

---

## plotEffects - [here](https://github.com/AvanishSalunke/statistics/commit/f612b6be4ffcf8d9b52ff143468831a83b636e89)

[plotEffects](https://in.mathworks.com/help/stats/linearmodel.ploteffects.html) shows how much each predictor moves the response, on its own, holding everything else fixed. It was already working on `LinearModel`, and it worked by going straight to the training data every time it was called: moving one predictor from its lowest to its highest observed value, keeping every other predictor at its average, and reading off how much the fitted response changes.

That approach can't carry over to `CompactLinearModel` as-is, because a compact model deliberately throws away the training data to stay small. So instead of computing the answer live, `LinearModel` now works out the exact same contrast once, right when the model is fit, and stores it in a new hidden property called `EffectContrasts` - one row per predictor. This gets carried over automatically whenever `compact` is called, the same way properties like `TermsMatrix` already are.

`LinearModel.plotEffects` itself was changed to just read from `EffectContrasts` instead of recomputing it every time it's called - a pure refactor, so the numbers it draws are unchanged from before. `CompactLinearModel.plotEffects` is a new, separate implementation that reads the same cached contrast, matching how the rest of the package keeps each class's methods independent instead of sharing code across files.

```octave
load carsmall
mdl  = fitlm ([Weight, Horsepower], MPG, 'linear');
cmdl = compact (mdl);

plotEffects (mdl)     ## already worked
plotEffects (cmdl)    ## now works too, using the cached contrast
```

<img src="{{ '/assets/posts/plotEffects_compact.png' | relative_url }}" alt="plotEffects on CompactLinearModel" style="display: block; margin: 1.5rem auto; width: 80%; max-width: 80%; height: auto; border: 4px solid #ffc107; border-radius: 4px;" />



## plotInteraction - [here](https://github.com/AvanishSalunke/statistics/commit/93fd2e9fdf1a539507e7a8f8c8d521235a2e207b)

[plotInteraction](https://in.mathworks.com/help/stats/linearmodel.plotinteraction.html) shows whether two predictors interact, by comparing each one's effect on its own against its effect when the other predictor is held at a few different values. The same problem applied here: it needed the training data to compute those comparisons.

<p style="text-align: left;">The same fix was used. A second new hidden property, <code>InteractionContrasts</code>, stores the contrasts needed for every pair of predictors that appear together in the model, computed once at fit time and carried over by <code>compact</code> just like <code>EffectContrasts</code>. <code>CompactLinearModel.plotInteraction</code> reads from it directly.</p>

```octave
plotInteraction (cmdl, 'x1', 'x2')                ## effects mode, default
```

<img src="{{ '/assets/posts/plotInteraction_compact.png' | relative_url }}" alt="plotInteraction on CompactLinearModel" style="display: block; margin: 1.5rem auto; width: 80%; max-width: 80%; height: auto; border: 4px solid #ffc107; border-radius: 4px;" />
## anova on LinearModel - [here](https://github.com/AvanishSalunke/statistics/commit/1d36678ab9aad372fe579a28b39fe89d87a6af80)

`anova` answers a simple question: out of everything in the model, how much is each individual predictor actually pulling its weight? It takes a fitted model and returns a table that breaks the total variation in the data into one row per predictor, plus a leftover "Error" row for whatever the model couldn't explain.

```octave
octave:28> load carsmall
octave:29> mdl = fitlm ([Weight, Horsepower], MPG, 'linear');
octave:30> anova (mdl)
```

```text
ans =
  3x5 table

              SumSq     DF    MeanSq        F         pValue       
             _______    __    _______    _______    ___________    

    x1       645.868     1    645.868     39.042    1.35188e-08    
    x2       83.7784     1    83.7784    5.06431        0.02686    
    Error    1488.86    90    16.5429        NaN            NaN  
```

Each row shows how much that one predictor contributes on its own, along with a p-value telling you whether that contribution is likely to be real or just noise. A small p-value means that predictor is probably doing genuine work in the model.

There's also a summary view, which zooms out and shows the bigger picture instead of one row per predictor - how much the whole model explains versus what's left unexplained:

```octave
octave:44> anova (mdl, 'summary')
```

```text
ans =
  3x5 table

                 SumSq     DF    MeanSq       F         pValue       
                _______    __    _______    ______    ___________    

    Total       6004.76    92    65.2691       NaN            NaN    
    Model        4515.9     2    2257.95    136.49    5.57376e-28    
    Residual    1488.86    90    16.5429       NaN            NaN    
```

Under the hood there's more than one accepted way to measure "how much a predictor contributes," and `anova` supports the main ones MATLAB does, with one less common variant still on the to-do list rather than shipped half-working.

Worth a quick note on why this was written from scratch rather than reused: there's another `anova` already in the package, built for a different GSoC project, that also happens to accept a fitted model as input. But it's really a different tool underneath - it builds its own separate kind of object, not the plain table that `LinearModel.anova` is supposed to return. Since the two don't actually do the same job, the right move was a fresh implementation built specifically around how `LinearModel` stores its terms and coefficients, rather than bending an unrelated tool to fit.

---

## Development Branch

All the code is on this branch:
[gsoc-2026-linearmodel](https://github.com/AvanishSalunke/statistics/tree/gsoc-2026-linearmodel)

---

## Next Steps

- `anova` on `CompactLinearModel`.
- `step` and [stepwiselm](https://in.mathworks.com/help/stats/stepwiselm.html) - automated model selection by forward and backward stepping.

Stay tuned for the next update!
