---
layout: post
title: "GSoC 2026 : Final Evaluation"
date: 2026-08-20
description: "The final evaluation submission: CompactLinearModel completed, stepwiselm documented and demoed, demos added across fitlm and LinearModel, and a round of bug fixes, all bundled into five PRs covering everything since midterm."
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

<p class="intro"><span class="dropcap">L</span>ast week brought <code>stepwiselm</code> and the <code>step</code> method, <code>anova</code> on <a href="https://in.mathworks.com/help/stats/classreg.regr.compactlinearmodel.html">CompactLinearModel</a>, and Type 3 sum of squares. This week is the final evaluation itself, and the five PRs below are the actual submission - together they cover everything built since midterm, not just what happened in the last few days.</p>

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
  <p style="margin-bottom: 0.8rem; text-align: left;"><a href="{{ '/blog/stepwiselm-and-step-anova-on-compactlinearmodel-and-type-3-sum-of-squares/' | relative_url }}"><strong>Week 11</strong></a> was <code>stepwiselm</code>, the <code>step</code> method, <code>anova</code> on <code>CompactLinearModel</code>, and Type 3 sum of squares.</p>
  <p style="margin-bottom: 0.8rem; text-align: left;"><strong>This week</strong> was the final evaluation submission itself: five PRs bundling everything built since midterm into a state ready for release.</p>
</div>

---

Eleven weeks in, this project has gone from a class with no properties defined to something that's genuinely usable end to end. It's been a slow build, most of it method by method, week by week, without a lot of dramatic jumps in between. What sticks out looking back isn't any single feature, it's how much of this came down to just showing up every week and checking one more thing against MATLAB. That's really the whole job, more than anything else.

For the final evaluation, the requirement from my mentor was the same as it was for midterm: get everything in a state that's ready to go into a proper package release, so it isn't just sitting on a development branch. Rather than trickling out as separate small PRs the way most weeks went, the post-midterm work got consolidated into five PRs that were submitted together as the actual final evaluation deliverable. Between them they cover the full second half of the project: finishing `CompactLinearModel`, documenting and demoing `stepwiselm`, adding demos across `fitlm` and every `LinearModel` method, and fixing the bugs that had built up along the way.

## Improving LinearModel and fixing bugs - [here](https://github.com/gnu-octave/statistics/pull/453)

[PR #453](https://github.com/gnu-octave/statistics/pull/453) covers the accumulated improvements and bug fixes made to `LinearModel` since midterm. None of it changes what the class does from the outside. It's the difference between a method working on the cases it was originally written against and holding up against everything MATLAB actually does with it.

## Completing CompactLinearModel - [here](https://github.com/gnu-octave/statistics/pull/455)

[PR #455](https://github.com/gnu-octave/statistics/pull/455) is where `CompactLinearModel` gets finished as a class. Every method that exists on `LinearModel` now has a matching, fully documented counterpart here, and the property documentation across the class was written out properly instead of being left half done. This was the single largest piece of post-midterm work, since most of `CompactLinearModel` was built in this stretch.

## stepwiselm documentation and demos - [here](https://github.com/gnu-octave/statistics/pull/456)

[PR #456](https://github.com/gnu-octave/statistics/pull/456) brings `stepwiselm` up to the same documentation standard as the rest of the package, full Texinfo docs covering every option, and demo blocks so it can be tried directly from the terminal instead of requiring someone to read the source first.

## Demos for fitlm - [here](https://github.com/gnu-octave/statistics/pull/454)

[PR #454](https://github.com/gnu-octave/statistics/pull/454) adds a full set of demo blocks to `fitlm` itself. Since `fitlm` is the entry point most people will actually use, this was about making sure it's properly demoed rather than left to speak for itself.

## Demos for every LinearModel method - [here](https://github.com/gnu-octave/statistics/pull/457)

[PR #457](https://github.com/gnu-octave/statistics/pull/457) does the same across the rest of `LinearModel`, every public method now has its own runnable demo. This was the widest of the five PRs just in terms of ground covered, going through the entire method list one at a time.

---

Around the same time, the statistics package put out its 1.9.0 release, and it's a big one. It's a major release built around the two GSoC 2026 projects, this one and the anova project, along with a large amount of additional MATLAB-parity work on top. The release brings in 113 new functions and classes altogether, spanning supervised learning, regression, clustering, dimensionality reduction, distributions, hypothesis testing, and plotting. `LinearModel`, `CompactLinearModel`, `stepwiselm`, and `fitlm` are all part of it, which means everything this project has built is no longer sitting on a development branch waiting to be useful. It's shipped, and anyone updating their installation gets it directly.

Sitting this close to the end of it, the thing that stands out most isn't any individual method or PR, it's that a piece of software I started poking at a year ago with no idea what I was doing is now something real people can install and use. That's not a small thing to sit with. The final evaluation period wraps up soon, and once it does, I'll be writing the full story of this GSoC, not a technical update like this one, but everything that led up to it.