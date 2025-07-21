---
description: >-
  A text analysis tool using NLP and topic modeling to uncover key feedback
  themes and enrollment drivers from open-ended survey responses.
---

# Topic Modeling - Cluster Analysis

## 🛠️ Overview

While working as a **Research Analyst for BYU’s Education Continuity Division**, I was tasked with helping the **Dean’s Office** better understand the **qualitative feedback** we were receiving from surveys sent to our continuing education program participants. Instead of manually reviewing hundreds of open-ended responses, I developed a **custom topic generation tool** using **Natural Language Processing (NLP)** and **unsupervised machine learning** to extract key themes from the data.

{% hint style="info" %}
The code snippets below are representative and do not show actual proprietary code.
{% endhint %}

***

## 🔥 Problem Statement

How can we **automatically extract the most common themes** from open-ended survey feedback, without manually reviewing each comment, in order to improve curriculum and student experience?

***

## 🧩 Project Pipeline

### 1. Data Collection and Preparation

We gathered over **1,000 survey comments** from post-course evaluation forms submitted by participants in various continuing education programs.

**Sources:**

* Internal Qualtrics survey exports
* Manual CSV extracts of comment data

**Cleaning and Transformation**

We used a standard NLP pipeline to prepare the data for modeling:

```r
rCopyEditlibrary(tm)
library(textclean)
library(SnowballC)

# Sample text vector
comments <- c("The exam did not reflect the material taught.",
              "Loved the instructor's teaching style.",
              "Customer service was not helpful.")

# Text cleaning
corpus <- VCorpus(VectorSource(comments))
corpus <- tm_map(corpus, content_transformer(tolower))
corpus <- tm_map(corpus, content_transformer(replace_contraction))
corpus <- tm_map(corpus, removePunctuation)
corpus <- tm_map(corpus, removeNumbers)
corpus <- tm_map(corpus, removeWords, stopwords("english"))
corpus <- tm_map(corpus, stemDocument)
corpus <- tm_map(corpus, stripWhitespace)
```

***

### 2. Topic Modeling with LDA

We then created a **Term Document Matrix (TDM)** and applied **Latent Dirichlet Allocation (LDA)** using the `topicmodels` package.

> **What is LDA?**
>
> LDA is an **unsupervised machine learning algorithm** used for **topic modeling** — a method to automatically discover the abstract “topics” that occur in a collection of documents. Each comment (or "document") is treated as a **mixture of topics**, and each topic is represented by a **distribution of words**.

In simpler terms, LDA groups together words that tend to appear in similar contexts, which allows us to **cluster comments** based on shared themes—even if they use different wording.

If you still have questions about this model, watch this 2 part video\
[https://www.youtube.com/watch?v=T05t-SqKArY](https://www.youtube.com/watch?v=T05t-SqKArY)

**Objective:**

Use unsupervised learning to group similar feedback comments by topic, reducing human effort and improving feedback interpretation.

**Methodology:**

* Create a Document-Term Matrix
* Run LDA with a selected number of topics
* Interpret word groupings manually

```r
library(topicmodels)

dtm <- DocumentTermMatrix(corpus)
dtm <- removeSparseTerms(dtm, 0.99)  # optional: remove sparse terms

lda_model <- LDA(dtm, k = 5, control = list(seed = 1234))

topics <- terms(lda_model, 5)
print(topics)
```

**Key Insights:**

* Each topic revealed a distinct feedback category
* Manual label mapping gave us interpretability
* Enabled quick review of **top 5 concerns**

### 3. Categorizing Comments by Topic

We assigned a most-likely topic to each comment to allow sorting and filtering by feedback themes.

```r
topic_assignments <- topics(lda_model)
head(topic_assignments)
```

This made the tool a highly practical solution for navigating qualitative feedback at scale.

***

## 📈 Results

Thanks to this tool, we identified the **top 5 feedback topics** participants cared about:

1. Exam not reflecting content
2. Teaching style
3. Customer service
4. Curriculum clarity
5. Technology/platform usability

The tool also helped identify the top 5 enrollment motivators for the various programs:

1. Program Requirement
2. Peer/Friend attending
3. Interest/relevance of content
4. Recommendation
5. Program or University reputation

These insights could serve as new key performance indicators to help program leads prioritize course improvements. The Dean’s Office was impressed and approved additional resources to expand the tool’s capabilities and integrate it into other programs.

***

## 🛤️ Future Improvements

If I were to continue developing this project, I would:

* Add a **Shiny dashboard** for end-user access and interaction
* Incorporate sentiment analysis using `syuzhet` or `textdata` packages
* Automate weekly ingestion of feedback via API connection to survey platform

***

Thanks for reading, and keep the learning going&#x20;
