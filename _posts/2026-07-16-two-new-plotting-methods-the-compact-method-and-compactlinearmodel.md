---
layout: post
title: "Two new plotting methods, the compact method, and CompactLinearModel"
date: 2026-07-16
description: "This week added the plot and plotInteraction methods, implemented the compact method, and introduced the CompactLinearModel classdef. The mentor also extracted four internal helpers into shared private functions."
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

<p class="intro"><span class="dropcap">T</span>his week two more plotting methods were added to <a href="https://in.mathworks.com/help/stats/linearmodel.html">LinearModel</a>: <code>plot</code> and <code>plotInteraction</code>. The <code>compact</code> method was also implemented, along with the initial <a href="https://in.mathworks.com/help/stats/classreg.regr.compactlinearmodel.html">CompactLinearModel</a> classdef. On the code organisation side, the mentor extracted four internal helper functions into shared private files so they can be reused across the package.</p>

## Project Status

<div style="text-align: left; margin: 1rem 0;">
  <p style="margin-bottom: 0.8rem; text-align: left;"><a href="{{ '/blog/introduction-and-gsoc-timeline/' | relative_url }}"><strong>Week 1</strong></a> was project planning and setup.</p>
  <p style="margin-bottom: 0.8rem; text-align: left;"><a href="{{ '/blog/starting-with-the-linearmodel-class/' | relative_url }}"><strong>Week 2</strong></a> was the class shell: properties, access rules, the display method, and the <code>subsref</code> gatekeeper.</p>
  <p style="margin-bottom: 0.8rem; text-align: left;"><a href="{{ '/blog/improving-fitlm-function-and-expanding-linearmodel-constructor/' | relative_url }}"><strong>Week 3</strong></a> was the constructor and <code>fitlm</code>: the math solver, all the diagnostics, the full 26-property object, and the front-end parser.</p>
  <p style="margin-bottom: 0.8rem; text-align: left;"><a href="{{ '/blog/adding-predict-refactoring-constructor/' | relative_url }}"><strong>Week 4</strong></a> was applying mentor feedback, refactoring the constructor, and adding <code>predict</code> and <code>random</code>.</p>
  <p style="margin-bottom: 0.8rem; text-align: left;"><a href="{{ '/blog/statistical-tests-autocorrelation-checks-and-model-modification/' | relative_url }}"><strong>Week 5</strong></a> was <code>feval</code>, <code>coefCI</code>, <code>coefTest</code>, <code>dwtest</code>, and <code>addTerms</code>.</p>
  <p style="margin-bottom: 0.8rem; text-align: left;"><a href="{{ '/blog/removeterms-three-new-plotting-methods-and-bug-fixes/' | relative_url }}"><strong>Week 6</strong></a> was <code>removeTerms</code>, <code>plotResiduals</code>, <code>plotDiagnostics</code>, <code>plotEffects</code>, and a set of bug fixes.</p>
  <p style="margin-bottom: 0.8rem; text-align: left;"><a href="{{ '/blog/midterm-evaluation-two-new-plotting-methods-and-robust-regression/' | relative_url }}"><strong>Week 7</strong></a> was the midterm evaluation, robust regression, <code>plotAdjustedResponse</code>, and <code>plotAdded</code>.</p>
  <p style="margin-bottom: 0.8rem; text-align: left;"><strong>This week</strong> was <code>plot</code>, <code>plotInteraction</code>, <code>compact</code>, and the initial <code>CompactLinearModel</code> structure.</p>
</div>

---

## Private Functions Extracted by the Mentor

Before this week's new methods were written, the mentor extracted four pieces of internal logic that were being used only inside `LinearModel` into standalone private functions. These now live in `inst/private/` and can be called by any class in the package:

- `build_design` - builds the design matrix from a terms matrix and an encoded predictor matrix
- `encode_categorical` - converts categorical columns into reference-coded dummy variables
- `parse_modelspec` - turns a modelspec keyword or string into a terms matrix
- `reencode_predictors` - re-encodes predictor data for prediction, handling categorical levels and power terms

<p style="text-align: left; margin-top: 1rem;"><code>LinearModel</code> now just calls these directly instead of carrying its own copies. Any future class that needs the same logic, like <code>GeneralizedLinearModel</code>, can call them too.</p>

## plot - [here](https://github.com/AvanishSalunke/statistics/commit/d501b900cc0ac3cf4a27ef9060e34d63d5bf2495)

[plot](https://in.mathworks.com/help/stats/linearmodel.plot.html) is a smart dispatcher. It looks at how many active predictors the model has and picks the most useful chart automatically.

```octave
plot (mdl)        ## picks the right plot automatically
h = plot (mdl)    ## get the graphics handles back
plot (ax, mdl)    ## target a specific axes object
```

There are three cases:

**No active predictors** - falls back to `plotResiduals` and shows the residual histogram.

**Exactly one predictor** - draws a scatter of the observed data with the fitted line and its 95% confidence bounds overlaid. For a categorical predictor, the x-axis shows the category levels.

**Two or more predictors** - delegates to `plotAdded` and shows the added variable plot for the whole model.

```octave
load carsmall
mdl1 = fitlm (Weight, MPG, 'linear');
plot (mdl1)     ## one predictor: scatter + fitted line + confidence bounds
```

<img src="{{ '/assets/posts/plot1.png' | relative_url }}" alt="plot - single predictor" style="display: block; margin: 1.5rem auto; width: 80%; max-width: 80%; height: auto; border: 4px solid #ffc107; border-radius: 4px;" />

```octave
mdl2 = fitlm ([Weight, Horsepower], MPG, 'linear');
plot (mdl2)     ## two predictors: delegates to plotAdded
```

<img src="{{ '/assets/posts/plot2.png' | relative_url }}" alt="plot - multiple predictors" style="display: block; margin: 1.5rem auto; width: 80%; max-width: 80%; height: auto; border: 4px solid #ffc107; border-radius: 4px;" />

---

## plotInteraction - [here](https://github.com/AvanishSalunke/statistics/commit/257d60d242498eb08f27fd4618d1cfae1866a750)

[plotInteraction](https://in.mathworks.com/help/stats/linearmodel.plotinteraction.html) shows how two predictors relate to each other in the model. You always pass two variable names, and optionally a plot type.

```octave
plotInteraction (mdl, 'x1', 'x2')               ## default: effects plot
plotInteraction (mdl, 'x1', 'x2', 'effects')    ## same as above
plotInteraction (mdl, 'x1', 'x2', 'predictions') ## predictions plot
h = plotInteraction (mdl, 'x1', 'x2')            ## get handles back
plotInteraction (ax, mdl, 'x1', 'x2')            ## into a specific axes
```

There are two modes.

**`'effects'` mode** (default) shows each predictor's effect as a dot with a 95% confidence interval line through it. It also shows conditional effects: what happens to one predictor's effect when the other is held at a few different values. If those conditional effect points sit at the same height as the main effect, the two predictors are not interacting. If they scatter at different heights, there is an interaction between them.

**`'predictions'` mode** plots the adjusted response as a curve across the range of the second predictor, with one curve per conditioning level of the first. An interaction shows up here as lines with different slopes.

```octave
load carsmall
mdl = fitlm ([Weight, Horsepower], MPG, 'interactions');
plotInteraction (mdl, 'x1', 'x2', 'effects')
```

<img src="{{ '/assets/posts/plotInteraction1.png' | relative_url }}" alt="plotInteraction effects mode" style="display: block; margin: 1.5rem auto; width: 80%; max-width: 80%; height: auto; border: 4px solid #ffc107; border-radius: 4px;" />

```octave
plotInteraction (mdl, 'x1', 'x2', 'predictions')
```

<img src="{{ '/assets/posts/plotInteraction2.png' | relative_url }}" alt="plotInteraction predictions mode" style="display: block; margin: 1.5rem auto; width: 80%; max-width: 80%; height: auto; border: 4px solid #ffc107; border-radius: 4px;" />

---

## compact and CompactLinearModel - [here](https://github.com/AvanishSalunke/statistics/commit/d3acf2ba2cb4ac18de001377d2365d583611660a)

[compact](https://in.mathworks.com/help/stats/linearmodel.compact.html) converts a fitted `LinearModel` into a `CompactLinearModel`:

```octave
cmdl = compact (mdl)
```

The idea behind a compact model is to keep only what you need after fitting - the coefficients, formula, model criteria, predictor names, and terms structure - and drop the training data entirely. This makes the object much smaller in memory, which is useful when saving a model to disk or deploying it without the original dataset.

`CompactLinearModel` is a new class added this week. What is implemented so far is the class structure: all the properties with full documentation, the `disp` and `display` methods, the `subsref` gatekeeper, and the constructor that copies the relevant properties across from a `LinearModel`.

```octave
cmdl = compact (mdl);

cmdl.Coefficients      ## works - copied from the original model
cmdl.Formula           ## works
cmdl.Rsquared          ## works
cmdl.ModelCriterion    ## works

cmdl.Fitted            ## error: unknown property
cmdl.Residuals         ## error: unknown property
cmdl.ObservationInfo   ## error: unknown property
```

Properties that require the original training data are not present on the compact object at all. Trying to access them throws an error immediately.

The display already works and matches the `LinearModel` display format:

```octave
load carsmall
mdl  = fitlm ([Weight, Horsepower], MPG, 'linear');
cmdl = compact (mdl)
```

```
cmdl =

  Compact linear regression model:
      y ~ 1 + x1 + x2

  Coefficients:

  3x4 table

                    Estimate         SE         tStat        pValue       
                   ___________    _________    ________    ___________    

    (Intercept)        47.7694      1.74166     27.4275    1.75104e-45    
    x1             -0.00656513    0.0010507    -6.24836    1.35188e-08    
    x2              -0.0420178    0.0186713     -2.2504        0.02686    


Number of observations: 93, Error degrees of freedom: 90
Root Mean Squared Error: 4.0673
R-squared: 0.752053,  Adjusted R-Squared: 0.746543
```

The methods like `predict`, `feval`, `coefCI`, and `coefTest` will be added on top of this structure next week.

---

## Development Branch

All the code is on this branch:
[gsoc-2026-linearmodel](https://github.com/AvanishSalunke/statistics/tree/gsoc-2026-linearmodel)

---

## Next Steps

- Implementing `predict`, `feval`, `random`, `coefCI`, and `coefTest` on `CompactLinearModel`.
- [stepwiselm](https://in.mathworks.com/help/stats/stepwiselm.html) - automated model selection by forward and backward stepping.

Stay tuned for the next update!
