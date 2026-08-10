---
layout: post
title: "stepwiselm and step, anova on CompactLinearModel, and Type 3 sum of squares"
date: 2026-08-10
description: "This week brought stepwiselm and the step method for automated model selection, the anova method for CompactLinearModel, and Type 3 sum of squares support for anova on LinearModel."
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

<p class="intro"><span class="dropcap">L</span>ast week brought <code>plotEffects</code> and <code>plotInteraction</code> to <a href="https://in.mathworks.com/help/stats/classreg.regr.compactlinearmodel.html">CompactLinearModel</a>, plus the <code>anova</code> method on <code>LinearModel</code>. This week two big pieces landed: <a href="https://in.mathworks.com/help/stats/stepwiselm.html">stepwiselm</a> and its companion <code>step</code> method, for automated model selection, along with <code>anova</code> on <code>CompactLinearModel</code> and Type 3 sum of squares support for <code>anova</code> on <code>LinearModel</code>.</p>

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
  <p style="margin-bottom: 0.8rem; text-align: left;"><a href="{{ '/blog/ploteffects-and-plotinteraction-on-compactlinearmodel-and-the-anova-method/' | relative_url }}"><strong>Week 10</strong></a> was <code>plotEffects</code> and <code>plotInteraction</code> on <code>CompactLinearModel</code>, and the <code>anova</code> method on <code>LinearModel</code>.</p>
  <p style="margin-bottom: 0.8rem; text-align: left;"><strong>This week</strong> was <code>stepwiselm</code>, the <code>step</code> method, <code>anova</code> on <code>CompactLinearModel</code>, and Type 3 sum of squares.</p>
</div>

---

## stepwiselm and step - [here](https://github.com/AvanishSalunke/statistics/commit/b4307c5b945dcddf189fcd43df800c1ca571bd26)

[stepwiselm](https://in.mathworks.com/help/stats/stepwiselm.html) builds a model automatically instead of you choosing the terms by hand. It starts from a simple model (a constant, by default) and keeps checking: is there a term outside the model that would clearly help if added? Is there a term inside the model that isn't pulling its weight and should be dropped? It keeps adding and removing one term at a time until neither move helps anymore.

```octave
octave:3> h1 = [7;1;11;11;7;11;3;1;2;21;1;11;10];
octave:4> h2 = [26;29;56;31;52;55;71;31;54;47;40;66;68];
octave:5> h3 = [6;15;8;8;6;9;17;22;18;4;23;9;8];
octave:6> h4 = [60;52;20;47;33;22;6;44;22;26;34;12;12];
octave:7> y  = [78.5;74.3;104.3;87.6;95.9;109.2;102.7;72.5;93.1;115.9;83.8;113.3;109.4];
octave:8> mdl = stepwiselm ([h1, h2, h3, h4], y, 'PEnter', 0.06)
1. Adding x4, FStat = 22.7985, pValue = 0.0005762318
2. Adding x1, FStat = 108.224, pValue = 1.105281e-06
3. Adding x2, FStat = 5.02586, pValue = 0.05168735
4. Removing x4, FStat = 1.86326, pValue = 0.2053954
mdl =

  Linear regression model:
      y ~ 1 + x1 + x2

  Estimated Coefficients:

  3x4 table

                   Estimate       SE         tStat       pValue       
                   ________    _________    _______    ___________    

    (Intercept)     52.5773      2.28617     22.998    5.45657e-10    
    x1              1.46831     0.121301    12.1047    2.69221e-07    
    x2              0.66225    0.0458547    14.4424    5.02896e-08    


Number of observations: 13, Error degrees of freedom: 10
Root Mean Squared Error: 2.40634
R-squared: 0.978678,  Adjusted R-Squared: 0.974414
F-statistic vs. constant model: 229.504, p-value = 4.40658e-09
```

You start with four candidate predictors and no idea which ones matter. `stepwiselm` doesn't just pick the two best predictors and stop - it works one step at a time, and each step only knows what happened before it. First x4 looks like the best single addition, so it goes in. Next, with x4 already in the model, x1 turns out to help the most, so it's added too. At that point x2 still clears the bar (barely, at PEnter = 0.06) and gets added as well. But now that all three are in, x4 isn't earning its place anymore - once x1 and x2 are both present, x4 stops being useful and its p-value crosses the removal threshold, so it gets dropped in the last step.

The end result only keeps x1 and x2, even though x4 was the very first term picked. That's the point of doing this step by step instead of just testing all combinations up front: a term can look good early on and still get removed later once better predictors are in the model alongside it.

Every step is judged by a chosen criterion - by default it's a p-value from comparing the model with and without the candidate term, but you can switch to AIC, BIC, R-squared, or adjusted R-squared instead. Categorical predictors are always added or removed as a whole group, never one dummy column at a time, and a higher-order term is never added unless its simpler building blocks are already in the model - the same logic MATLAB follows.

The `step` method does the same search, but starting from a model you've already fit, and by default takes just one step instead of running to completion:

```octave
octave:12> mdl0 = fitlm ([h1, h2, h3, h4], y, 'y ~ 1 + x1')
mdl0 =

  Linear regression model:
      y ~ 1 + x1

  Estimated Coefficients:

  2x4 table

                   Estimate       SE        tStat       pValue       
                   ________    ________    _______    ___________    

    (Intercept)     81.4793     4.92734    16.5362    4.06707e-09    
    x1              1.86875    0.526407       3.55     0.00455204    


Number of observations: 13, Error degrees of freedom: 11
Root Mean Squared Error: 10.7267
R-squared: 0.533948,  Adjusted R-Squared: 0.49158
F-statistic vs. constant model: 12.6025, p-value = 0.00455204
octave:13> mdl = step (mdl0)
1. Adding x2, FStat = 208.582, pValue = 5.02896e-08
mdl =

  Linear regression model:
      y ~ 1 + x1 + x2

  Estimated Coefficients:

  3x4 table

                   Estimate       SE         tStat       pValue       
                   ________    _________    _______    ___________    

    (Intercept)     52.5773      2.28617     22.998    5.45657e-10    
    x1              1.46831     0.121301    12.1047    2.69221e-07    
    x2              0.66225    0.0458547    14.4424    5.02896e-08    


Number of observations: 13, Error degrees of freedom: 10
Root Mean Squared Error: 2.40634
R-squared: 0.978678,  Adjusted R-Squared: 0.974414
F-statistic vs. constant model: 229.504, p-value = 4.40658e-09
```

`step(mdl0)` takes exactly one step and adds x2, since it's the strongest candidate. `step(mdl0, 'NSteps', 3)` is allowed to take up to three steps, but it stops after just one anyway - once x2 is in, neither x3 nor x4 is good enough to add, and nothing in the model is weak enough to remove. So both calls land on the same model here; NSteps only matters when there's still more room to improve.

## anova on CompactLinearModel - [here](https://github.com/AvanishSalunke/statistics/commit/1dad3555a594468c9f2255d4fbf0b01f207a4389)

<p style="text-align: left;"><code>anova</code> tells you how much each predictor is pulling its weight in the model. It already worked on <code>LinearModel</code>, and now it works on <code>CompactLinearModel</code> too, with the same output.</p>

```octave
octave:17> load carsmall
mdl  = fitlm ([Weight, Horsepower], MPG, 'linear');
cmdl = compact (mdl);
octave:20> anova (cmdl)
ans =
  3x5 table

              SumSq     DF    MeanSq        F         pValue       
             _______    __    _______    _______    ___________    

    x1       645.868     1    645.868     39.042    1.35188e-08    
    x2       83.7784     1    83.7784    5.06431        0.02686    
    Error    1488.86    90    16.5429        NaN            NaN    
```

You can also ask for the bigger-picture view instead:

```octave
octave:21> anova (cmdl, 'summary')
ans =
  3x5 table

                 SumSq     DF    MeanSq       F         pValue       
                _______    __    _______    ______    ___________    

    Total       6004.76    92    65.2691       NaN            NaN    
    Model        4515.9     2    2257.95    136.49    5.57376e-28    
    Residual    1488.86    90    16.5429       NaN            NaN    
```

Same idea, just zoomed out - how much the whole model explains versus what's left over.

## Type 3 Sum of Squares - [here](https://github.com/AvanishSalunke/statistics/commit/4934ef0014d872679e917895605c7696fa9980e9)

There's more than one way to measure "how much does this predictor matter." Until now `anova` only offered one way. Now you can ask for Type 3 instead:

```octave
octave:27> anova (mdl, 'components', 3)
ans =
  3x5 table

              SumSq     DF    MeanSq        F         pValue       
             _______    __    _______    _______    ___________    

    x1       645.868     1    645.868     39.042    1.35188e-08    
    x2       83.7784     1    83.7784    5.06431        0.02686    
    Error    1488.86    90    16.5429        NaN            NaN    
```

Type 3 asks: if I pulled just this one term out and kept everything else exactly as it is, how much would the model suffer? It's a stricter, more isolated way of judging each term compared to the default.

---

## Development Branch

All the code is on this branch:
[gsoc-2026-linearmodel](https://github.com/AvanishSalunke/statistics/tree/gsoc-2026-linearmodel)

---

## Next Steps

- Improving documentation across the newer methods.
- Adding more tests to cover edge cases.
- Adding demo blocks so the new methods can be tried straight from the terminal.
- Merging in the latest round of mentor changes and feedback.

Stay tuned for the next update!
