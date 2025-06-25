---
description: Reviving a Unified Satisfaction Survey for Education Programs
---

# Satisfaction Survey Analysis

## 🛠️ Overview

The satisfaction survey for the Continuing Education department had been discontinued due to its inability to provide consistent, actionable insights across a diverse range of programs. This project brought it back to life by identifying key perfomance indicators and condensing the survey structure using various regression analysis techniques.

{% hint style="info" %}
_Code snippets serve only as visual aids to represent the code used in the project. The actual code generated in this project is not shared for intellectual property reasons._
{% endhint %}

***

## 🔥 Problem Statement

The Continuing Education department sought to measure satisfaction across various events and programs using a single unified survey. However, diverse goals, audiences, and KPIs across programs made it difficult to compare results meaningfully. The objective was to identify satisfaction drivers that were consistent across programs and to update the survey to reflect these common factors.

***

## 🧩 Project Pipeline

### 1. Survey Data Collection & Cleaning

The original survey had two parts: a standardized set of 16 questions for all programs and a second section tailored per program. Questions included demographic information, satisfaction, perceived value, cost, expectations, customer service, facilities, and open-ended feedback.

**Sources:**

* Qualtrics survey data inlcluding:
  * Five years of survey responses
  * Metadata such as completion rate and time spent per respondent

**Cleaning and Transformation:**

* Removed duplicates and entries lacking satisfaction scores
* Filtered for entries with at least 80% completion
* Ensured reliable survey responses (avg. time spent was 1.6 to 2 minutes)

> R Example

```r
# Load and filter the dataset
library(dplyr)

clean_data <- raw_data %>%
  filter(completion_rate >= 0.8, !is.na(satisfaction_score)) %>%
  distinct()
```

***

### 2. Exploratory Data Analysis

With the dataset cleaned, it's now both feasible and essential to explore the characteristics and relationships among the variables to inform the selection of an appropriate model.

**Objective:**

Perform an exploratory data analysis using statistical and visualization methods to uncover trends or insights that will help us choose the model.

**Methodology:**

*   Generate a statistical summary and visualized individual variables using histograms and box plots to assess distributions, central tendency, and variability.

    <figure><img src="../.gitbook/assets/Screen Shot 2025-05-28 at 10.47.14 AM.png" alt=""><figcaption><p>Histograms examples</p></figcaption></figure>

    * One numeric variables
    * Five categorical variables
    * Five ordinal variables
      * Ordinal variables were scaled from 1 to 5, where 1 indicates the most negative response and 5 the most positive.
    * The target variable, "Satisfaction," is ordinal.
* Construct a correlation heatmap to identify strong linear relationships between explanatory variables.

**Key Insights:**

High multicollinearity was detected among **Perceived Value**, **Cost**, and **Expectations Met**, which means there is redundancy between questions. To address this, I would need to do variable selection using a penalized regression method such as LASSO or Elastic Net.&#x20;

* This approach not only reduces the number of predictors used in the model but also has the added benefit of streamlining the survey by eliminating less informative questions—potentially improving the overall completion rate.

Given that the target variable "Satisfaction" is ordinal with more than two levels, a **multinomial logistic regression** is an appropriate modeling approach. This allows for modeling the probability of responses across multiple ordered satisfaction levels.

### 3. Variable Selection

Variable selection is the process of choosing the most important questions or factors from a larger list to simplify the analysis by focusing only on the inputs that matter most.

**Objective:**

The goal of this step was to identify the most relevant predictors of satisfaction from a broader set of explanatory variables. Effective variable selection was crucial to improving model interpretability, addressing multicollinearity, and ultimately enhancing the performance and simplicity of the multinomial logistic regression model.

**Methodology:**

* Treat target variable as numeric just for feature selection.
* Apply a suite of **regularized linear regression techniques t**o select the most informative features, including:
  * **Ridge Regression** (L2 penalty): To identify variables with weaker influence while retaining all predictors.
  * **LASSO Regression** (L1 penalty): To shrink less important coefficients to zero, effectively performing variable selection.
  * **Elastic Net** (combination of L1 and L2 penalties): To balance between Ridge and LASSO, especially useful when predictors are highly correlated.
* Compare the variable selection results across all three methods to identify consistent patterns in which variables were retained or eliminated.

This comprehensive approach allowed for a robust evaluation of variable importance, accounting for both statistical strength and redundancy.

> R Code (Example)

```r
# Load libraries
library(glmnet)
library(caret)

# Convert categorical predictors to dummy variables using model.matrix
# Assume your data frame is called survey_data
# and 'Satisfaction' is the target variable (ordinal 1-5)

# Convert target to numeric
y <- as.numeric(survey_data$Satisfaction)

# Create design matrix for predictors (excluding the intercept)
x <- model.matrix(Satisfaction ~ . - 1, data = survey_data)

# LASSO (L1 Penalty)
set.seed(123)
cv_lasso <- cv.glmnet(x, y, alpha = 1)
plot(cv_lasso)
coef(cv_lasso, s = "lambda.min")  # View selected variables

# Ridge (L2 Penalty)
cv_ridge <- cv.glmnet(x, y, alpha = 0)
plot(cv_ridge)
coef(cv_ridge, s = "lambda.min")

# Elastic Net (Mixed L1/L2)
cv_enet <- cv.glmnet(x, y, alpha = 0.5)  # alpha = 0.5 is a mix of L1 and L2
plot(cv_enet)
coef(cv_enet, s = "lambda.min")

# Extract variables with non-zero coefficients
selected_vars <- coef(cv_lasso, s = "lambda.min")
selected_vars <- selected_vars[selected_vars[, 1] != 0, , drop = FALSE]
print(selected_vars)
```

**Key insights:**

Across all three methods, **Perceived Value** and **Cost** consistently had coefficients near zero or were entirely excluded, suggesting they provided limited unique predictive value in the presence of other variables. In contrast, **Expectations Met** retained a stronger coefficient, indicating it held more explanatory power.

As a result, **Perceived Value** and **Cost** would be excluded from the final model to reduce redundancy and improve interpretability.&#x20;

### 4. Identifying Key Drivers of Satisfaction (Multimonial Regression)

With a cleaned dataset and a set of non-redundant variables, we can now build our final model to analyze what influences satisfaction the most.

**Objective:**

In this step we will identify which factors most strongly influence satisfaction across different programs, while accounting for demographic differences (such as role) and program-specific characteristics.

**Methodology:**&#x20;

* Conducted a multinomial regression analysis to model satisfaction outcomes
* Evaluated p-values and coefficient estimates to assess variable significance
* Interpreted results to identify key drivers of satisfaction

> R Example (Multinomial Logistic Regression)

```r
# Multinomial Logistic Regression
library(nnet)

multi_model <- multinom(as.factor(Satisfaction) ~ age + gender + program + expectations_score +
                        customer_service_score + role + reason_enrolled + program + facilities_score, data = survey_data)
summary(multi_model)

# Get z-values and p-values
z <- summary(model)$coefficients / summary(model)$standard.errors
p_values <- 2 * (1 - pnorm(abs(z)))

# Print p-values
print(p_values)

# Print coefficients
print(coef(model))
```

**Key Insights:**

* **Cost** was not a statistically significant predictor of satisfaction
* **Customer service** and whether **expectations were met** were strong predictors of positive satisfaction outcomes
* **Role differences** emerged: parents tended to rate their experiences worse than direct participants
* The **reason for enrollment** (e.g., personal interest,  career advancement, etc.) had a notable impact on satisfaction levels

### 5. Survey Redesign Based on Data

After identifying which variables had the most predictive power and were generalizable across programs, I worked with the team to redesign the survey.

**Objective:**\
Create a leaner, more effective survey for consistent satisfaction benchmarking.

**Methodology:**

* Reduced the questionnaire to just 7 core questions
  1. Program
  2. Role (Parent vs Participant)
  3. Reason for enrolling
  4. Expectations met
  5. Customer service
  6. Overall satisfaction
  7. Open-ended feedback

The revised survey design not only reduced respondent burden but also preserved the most impactful predictors of satisfaction, enabling more consistent and actionable benchmarking across programs.

***

## 📈 Results

The analysis and redesign effort led to the revival of a unified, data-informed satisfaction survey. Key outcomes included:

* A streamlined survey that improved clarity and reduced average completion time by approximately 50%
* An estimated 30–40% increase in completion rate, based on industry benchmarks for shorter surveys
* Clear identification of department-wide KPIs: **Customer Service & Expectations Met**
* Executive buy-in following a strategic presentation to the Dean and Board
* A scalable foundation for ongoing root-cause analysis and satisfaction tracking

Although I departed before full implementation, the department accepted the proposal and tasked my team with deployment. The final survey was concise, aligned with key metrics, and designed to support long-term program evaluation.

***

## 🛤️ Future Improvements

To strengthen the value and impact of the satisfaction survey, here are a few possible enhancements:

**1. Enhance Multinomial Model**

* Explore interaction effects between satisfaction drivers and program type to uncover more nuanced, program-specific insights.
* Evaluate the survey’s ongoing performance, revisiting predictive validity and model assumptions to ensure continued relevance.

**2. Expand Text Analytics for Deeper Insight**

* Continue leveraging natural language processing (NLP), including topic modeling and sentiment analysis, to uncover root causes behind satisfaction trends.

**3. Build Reporting Tool for Ongoing Monitoring**

* Build interactive dashboards or automated reports to empower department leads to insights for timely decision-making.

***

Identifying gaps in satisfaction is not just about scores—it's about understanding context, behavior, and expectations.&#x20;

Thanks for reading, and keep the learning going.
