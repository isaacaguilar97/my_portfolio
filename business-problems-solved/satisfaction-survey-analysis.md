---
description: Reviving a Unified Satisfaction Survey for Continuing Education
---

# Satisfaction Survey Analysis

## 🛠️ Overview

The satisfaction survey for the Continuing Education department had been discontinued due to its inability to provide consistent, actionable insights across a diverse range of programs. This project brought it back to life by redesigning the analysis methodology and condensing the survey structure without losing the depth of feedback using various regression analysis techniques.

{% hint style="info" %}
_Code snippets serve only as visual aids to represent the code used in the project. The actual code generated in this project are not shared for intellectual property reasons._
{% endhint %}

***

## 🔥 Problem Statement

The Continuing Education department sought to measure satisfaction across various events and programs using a single unified survey. However, diverse goals, audiences, and KPIs across programs made it difficult to compare results meaningfully. The challenge was to transform the survey and analysis methodology in a way that honored those differences while still producing comparable satisfaction metrics.

***

## 🧩 Project Pipeline

### 1. Survey Data Collection & Cleaning

The original survey had two parts: a standardized set of 16 questions for all programs and a second section customized per program. Questions included demographic information, satisfaction, perceived value, cost, customer service, facilities, and open-ended feedback.

**Sources:**

* Qualtrics survey data brough in via API inlcluding:
  * Five years of survey responses
  * Metadata such as completion rate and time spent per respondent

**Cleaning and Transformation:**

* Removed duplicates and entries lacking satisfaction scores
* Filtered for entries with at least 80% completion
* Ensured reliable survey responses (avg. time spent was 1.6 to 2 minutes)
* Transformed ordinal data to numeric for modeling

> R Example

```r
# Load and filter the dataset
library(dplyr)

clean_data <- raw_data %>%
  filter(completion_rate >= 0.8, !is.na(satisfaction_score)) %>%
  distinct()
```

***

### 2. Identifying Key Drivers of Satisfaction

With a cleaned dataset, I explored the relationships between variables to find which truly influenced satisfaction across programs.

**Objective:**\
Determine the most influential variables in satisfaction, accounting for demographic and program-specific differences.

**Methodology:**

* Performed exploratory data analysis to understand data structure and relationships
* Noted multicollinearity (e.g., value highly correlated with satisfaction)
* Applied linear regression, Ridge, LASSO, and Elastic Net for variable selection
* Used Multinomial Logistic Regression for ordinal satisfaction outcomes
* Modeled satisfaction with independent variables such as:
  * Age
  * Gender
  * Role (participant or parent)
  * Value, Customer Service, Reason for Enrolling
  * Program attended

**Key Insights:**

* Cost was not a significant factor in satisfaction
* Customer service and perceived value were highly predictive
* Role mattered—parents rated experiences differently than participants
* The reason for enrollment also influenced satisfaction

> R Example (Variable Selection)

```r
# Transform ordinal variables to numeric
survey_data <- clean_data %>%
  mutate(across(c(value_score, service_score, satisfaction_score), as.numeric))

# LASSO Regression
library(glmnet)

X <- model.matrix(satisfaction_score ~ age + gender + value_score + service_score + 
                  role + reason_enrolled + program, data = survey_data)[, -1]
y <- survey_data$satisfaction_score

lasso_model <- cv.glmnet(X, y, alpha = 1)  # LASSO

# Ridge Regression (alpha = 0)
ridge_model <- cv.glmnet(X, y, alpha = 0)

# Elastic Net (alpha between 0 and 1)
enet_model <- cv.glmnet(X, y, alpha = 0.5)
```

> R Example (Multinomial Logistic Regression)

```r
# Multinomial Logistic Regression
library(nnet)

multi_model <- multinom(as.factor(satisfaction_score) ~ age + gender + value_score +
                        service_score + role + reason_enrolled + program, data = survey_data)
summary(multi_model)
```

***

#### 3. Survey Redesign Based on Data

After identifying which variables had the most predictive power and were generalizable across programs, I worked with the team to redesign the survey.

**Objective:**\
Create a leaner, more effective survey for consistent satisfaction benchmarking.

**Methodology:**

* Reduced the questionnaire to just 5 core questions + demographics
  1. Overall satisfaction
  2. Expectations met
  3. Customer service
  4. Reason for enrolling
  5. Open-ended feedback
* Used topic modeling and sentiment analysis on feedback to complement quantitative scores
* Reduced average completion time by \~50%
* Estimated a 30–40% increase in response rate based on industry benchmarks for shorter surveys

> R Example (Topic Modeling)

```r
library(tm)
library(topicmodels)

# Clean and prep text
docs <- Corpus(VectorSource(survey_data$open_feedback))
docs <- tm_map(docs, content_transformer(tolower))
docs <- tm_map(docs, removePunctuation)
docs <- tm_map(docs, removeWords, stopwords("en"))

dtm <- DocumentTermMatrix(docs)
lda_model <- LDA(dtm, k = 5)  # 5 topics
topics <- terms(lda_model, 5)
```

**Example Improvements:**

* Programs now share comparable metrics without losing their unique context
* Satisfaction is now tied to generalizable factors like service and expectations rather than program-specific metrics

***

### 📈 Results

The analysis and redesign led to the resurrection of the unified survey, now streamlined and backed by predictive modeling. Key results included:

* A simplified yet powerful survey, improving response rate and clarity
* Clear identification of department-wide KPIs (Customer Service, Expectations Met, Value)
* Executive buy-in following a strategic presentation to the Dean and Board
* Actionable insights through topic modeling and demographic segmentation
* A scalable model for ongoing root-cause analysis and satisfaction tracking

Though I left the company before implementation, the department accepted the proposal and placed my team in charge of deployment.

***

### 🛤️ Future Improvements

**Enhance Comparability Across Time and Programs**

* Adjust satisfaction scores by demographic profile using normalization techniques
* Introduce confidence intervals for satisfaction scores per segment

**Expand Text Analytics**

* Continue leveraging NLP to identify patterns in qualitative feedback
* Build dashboards for department leads to interactively explore satisfaction trends

**Optimize Longitudinal Measurement**

* Monitor shifts in satisfaction over time by program type and delivery format (online, hybrid, in-person)

***

Identifying gaps in satisfaction is not just about scores—it's about understanding context, behavior, and expectations.&#x20;

Thanks for reading, and keep the learning going.
