---
description: How can we make the fluffiest pancakes?
---

# Pancakes' Study - ANOVA

## 🛠️ Overview

This fun and statistically grounded project was designed to determine which factors most significantly affect the height or "fluffiness" of pancakes. Using a full factorial experimental design, we tested three brands of complete pancake mix, two types of liquids, and two types of non-stick sprays to investigate their individual and interactive effects on pancake height.

This project combined hands-on experimentation with statistical modeling using R. It was conducted as part of a collaborative exploration into real-world applications of factorial ANOVA.

***

## 🔥 Problem Statement

We sought to answer:

* Does the brand of pancake mix influence fluffiness?
* Does using milk instead of water make pancakes thicker?
* Does the type of non-stick spray used make any difference?
* Are there significant interactions between these factors?

***

## 🧩 Project Pipeline

### 1. Data Collection

We collected primary data by physically preparing and measuring pancakes across combinations of:

* **Mix Brand**: Great Value, Betty Crocker, Hungry Jack
* **Liquid Type**: Water vs. Whole Milk
* **Non-Stick Spray**: Kroger vs. Pam

We used measuring tape and toothpicks to quantify height (in cm) as the dependent variable.

**Sources:**

* Direct experimentation
* Manual measurements
* Number of observations: 24
  * Number of groups (recipes): 12
  * Number of observations per group: 2

**Cleaning and Transformation:**

* Verified consistent measurements
* Encoded categorical variables

```r
Data <- read.csv("pancake_data.csv")
# Turn Data columns into factors
Brand <- as.factor(Data$Brand)
Nonstick <- as.factor(Data$Non.stick)
Liquid <- as.factor(Data$Liquid)
Size <- as.factor(Data$Size)
Height <- Data$Size.height..cm.
```

### 2. Exploratory Data Analysis

Before modeling, we explored the data visually to understand patterns and potential interactions. This step helps check for obvious group differences or the need for transformations.

```r
# Create box plots to compare means and review possible data transformation
boxplot(Height ~ Brand)
boxplot(Height ~ Liquid)
boxplot(Height ~ Nonstick)

# Review for possible interaction with Interaction plots
interaction.plot(Brand, Liquid, Height)
interaction.plot(Liquid, Brand, Height)
interaction.plot(Brand, Nonstick, Height)
interaction.plot(Nonstick, Brand, Height)
interaction.plot(Liquid, Nonstick, Height)
interaction.plot(Nonstick, Liquid, Height)
```

**Key Insight:**\
From the interaction plots, there appeared to be a possible interaction between **Brand** and **Non-stick Spray**—evidenced by crossing lines. This visual clue informed our decision to test for interactions formally in the ANOVA model.

<figure><img src=".gitbook/assets/Screen Shot 2025-07-15 at 7.01.04 AM.png" alt=""><figcaption></figcaption></figure>

### 3. Analysis of Variance (ANOVA)

**ANOVA** is a statistical method used to compare the means of multiple groups and determine whether any of them differ significantly from each other.&#x20;

**Objective:**

In this project, we used a **three-way factorial ANOVA**, which allows us to analyze:

* The **main effect** of each factor (brand, liquid, spray)
* The **interaction effects** between any pair or all three of these factors

This is particularly powerful when testing multiple variables at once, because it lets us test everything together instead of doing a bunch of separate comparisons, which could give misleading results.

**Methodology:**

We modeled the pancake height using the formula

<p align="center"><strong><code>Height ~ Brand * Liquid * Nonstick</code></strong></p>

This captures all the main effects and their interactions.

```r
model <- aov(Height ~ Brand * Liquid * Spray, data = Data)
summary(model)
```

<figure><img src=".gitbook/assets/Screen Shot 2025-07-15 at 7.18.02 AM.png" alt=""><figcaption></figcaption></figure>

**Key Insights:**

* No statistically significant main or interaction effects (p > 0.05)
* The suspected Brand x Nonstick interaction was not statistically validated
* Results may reflect underpowered study conditions rather than true lack of effect

***

### 4. Power Analysis

Before drawing conclusions from our ANOVA model, we evaluated how likely we were to detect real effects given our sample size—this is called **statistical power**. A power analysis tells us how many replicates we would need per treatment to have a strong chance (typically ≥80%) of detecting meaningful differences.

**Methodology**

We used the `power.anova.test()` function in R with the following inputs:

* **groups = 12**: Our full factorial design included 12 unique treatment combinations (3 Brands × 2 Liquids × 2 Non-stick sprays).
* **between.var**: The variance across the 12 group means, calculated using `var(c(...))`.
* **within.var = 0.037917**: The residual variance (also known as the Mean Square Error), estimated from our ANOVA model.
* **n = 2:8**: A range of possible sample sizes (replicates per group).
* **sig.level = 0.05**: Standard alpha level for significance.

```r
# Create Power Curve to determine how many observations did we needed to have a power bigger than .8
tapply(Height, Brand:Nonstick:Liquid, mean) # All my treatment means
i = power.anova.test(groups=12, between.var = var(c(1.6,1.45,1.45,1.4,1.8,1.45,1.85,1.6,1.55,1.3,1.65,1.75)), within.var = variance, sig.level = .05, n=c(2:8))
x=2:8
plot(x,i$power, xlab = "Number of replications", ylab = 'Power to detect differences', type = "l") # We should have done 3 replications per treatment (36 observations) to have a power of .817
```

<figure><img src=".gitbook/assets/Screen Shot 2025-07-15 at 7.32.29 AM.png" alt=""><figcaption></figcaption></figure>

**Key Insights:**

* With only **2 replicates per group**, our statistical power was **\~0.45**, meaning we had less than a 50% chance of detecting a true effect.
* According to the power curve, we would have needed **at least 3 replicates per treatment** (total of 36 observations) to achieve a power of **\~0.82**, which meets the conventional threshold of 80%.

### 5. Presentation

My peers and I shared our findings in our Statistics class&#x20;

{% embed url="https://docs.google.com/presentation/d/1EuZWp4EmA4bxDL6zMh7TfzU5P8PYsgdI9MvoOumfj90/edit?usp=sharing" %}

***

## 📈 Results

Our study was **underpowered**, which likely contributed to our inability to detect statistically significant effects, failing to reject the null hypothesis

However, the experiment provided a solid foundation for learning about factorial design and model assumptions.

***

## 🛤️ Future Improvements

* **Increase sample size** for greater statistical power
* Consider blind height measurements to reduce bias

***

Thanks for reading! This was a delicious experiment in both data science and breakfast food. Whether or not we found significance, we cooked up curiosity—and that's always worth it.&#x20;

Thanks for reading, and keep the learning going.
