---
layout: post
title:  "Starting with the LinearModel class"
date:   2026-06-05
description: "Establish core LinearModel architecture: define protected public/private properties, override display methods for formatted statistical output, and implement custom subscripted reference method."
---

<p class="intro"><span class="dropcap">B</span>efore jumping straight into the matrix algebra, I spent this week focusing on data security, command-line display, and how this class will handle user inputs. Here is a look at the architecture we established.</p>

## Locking Down the Data

A fitted [LinearModel](https://in.mathworks.com/help/stats/linearmodel.html) holds dozens of statistical properties like mean squared error, coefficients, and residuals. The main challenge is ensuring users can easily read these properties but cannot accidentally overwrite them. For example, we do not want a user typing `mdl.MSE = 0` in the command window and breaking the model's integrity.

Instead of writing a complicated custom [subscripted assignment](https://in.mathworks.com/help/fixedpoint/ref/embedded.fi.subsasgn.html) method to intercept user inputs, I relied on Octave’s native access modifiers:

```octave
properties (GetAccess = public, SetAccess = protected)
```

This prevents users from tampering with the statistics, while still allowing the class constructor to populate the data cleanly when it initializes. I also created a hidden block of properties to securely store the raw data matrices. This ensures that later on, when a user wants to make predictions on new data, the model always has an uncorrupted baseline to work from without cluttering the user's workspace.

## Cleaning Up the Output and Indexing

By default, if you try to print a massive object in Octave, you get an unreadable wall of text dumping every single piece of data to the screen.

To fix this, I overrode the standard display methods. The object now uses defensive programming to check which properties have actually been calculated and prints a clean, formatted statistical summary. For the actual model coefficients, I passed the data directly to Octave’s native `table` class, which handles all the complex column alignment and formatting for us.

I also wrote a custom [subscripted reference](https://in.mathworks.com/help/matlab/ref/subsref.html) method. This acts as a strict gatekeeper. It forces users to use standard dot notation (like `mdl.MSE`), explicitly blocks illegal array indexing (like `mdl(1)`), and makes sure legitimate method calls are routed correctly to the Octave engine.

## Plan for Next Week: Internal Private Methods

Now that the secure container is built, my plan for next week is to start writing the actual mathematics. I will be building a set of private helper methods directly inside the LinearModel class. Breaking the math down into these specific internal methods keeps the code organized, easy to read, and much easier to test.

Here is what I will be implementing in detail:

- **`lm_fit` (The Core Solver)**: This is where the actual regression happens. Instead of using standard equations that can be unstable, this method will take the numeric matrices and use a technique called QR decomposition to calculate the regression coefficients. It will also have built-in checks to safely handle any redundant or highly correlated data columns without crashing.
- **`lm_diagnostics` (Outlier Detection)**: This method will run immediately after the model is fitted. Its job is to calculate metrics like Leverage and Cook's Distance. Essentially, it sweeps through the data row by row to see if any single data point is extreme enough to unfairly skew the final results.
- **`lm_predict` (Making Forecasts)**: A dedicated method to apply the finished model to brand new data. It will not just spit out a single predicted number; it will calculate the exact confidence margins, letting the user know the potential error range of the prediction.
- **`lm_encode_categorical` (Data Translation)**: Matrix algebra only understands numbers. If a user passes a dataset with a text column (like car colors: "Red" or "Blue"), the solver will fail. This method translates those text categories into binary dummy variables (columns of 0s and 1s) so the math can process them smoothly.
- **`lm_parse_modelspec` (Formula Parsing)**: This method bridges the gap between human input and the math solver. It takes human-readable formulas, like `'MPG ~ Weight * Acceleration'`, and translates them into a precise numeric matrix. This matrix acts as a blueprint, telling the solver exactly which columns to multiply together.

## GSoC Development Branch

All the development work and code changes for this project are being actively tracked in my dedicated GNU Octave statistics package branch: [gsoc-2026-linearmodel](https://github.com/AvanishSalunke/statistics/tree/gsoc-2026-linearmodel). 

Over the coming weeks, this branch will host the direct implementation of the LinearModel class, its methods, and associated validation tests. Keeping the work isolated in this branch allows for continuous testing and review before it eventually gets merged into the official Octave repository.

## Next Steps

Once those five internal methods are implemented next week, the final step for this phase will be writing the `fitlm` function. The `fitlm` function will act as the front-end parser to clean out missing values and format the raw data before passing it to the LinearModel constructor.

Stay tuned for the next update, where we finally dive into the matrix algebra and get the solver working!
