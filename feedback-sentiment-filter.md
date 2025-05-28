# Feedback Sentiment Filter

## 🛠️ Overview

At BYU Continuing Education, open-ended feedback from student satisfaction surveys often went unread by business departments due to the overwhelming volume and time commitment required. To bridge this gap, I developed an **interactive feedback sentiment filter**—a tool that uses machine learning to automatically classify comment sentiment and organize it by program, respondent demographics, and key topics. The goal was to make qualitative feedback actionable, enabling faster, more targeted analysis to inform program improvements.

> _Note: Code snippets serve only as visual aids to represent the code used in the project. The actual code generated in this project is not shared for intellectual property reasons._

***

## 🔥 Problem Statement

How can we make years of qualitative feedback more accessible, meaningful, and actionable without overwhelming department staff?\
How can we automate sentiment classification without sacrificing accuracy or nuance?

***

## 🧩 Project Pipeline

### 1. Data Collection & Preprocessing

I began by aggregating five years of survey responses from BYU Continuing Education. These surveys contained both structured data (e.g., program type, role, demographics) and unstructured text responses.

**Sources:**

* Internal survey database
* Demographic metadata (role, age, program, etc.)

**Cleaning and Transformation:**

* Removed duplicates, irrelevant or empty responses
* Standardized text (lowercasing, punctuation removal, etc.)
* Filtered for English-language responses
* Balanced dataset for sentiment classification (positive, neutral, negative)

**Code Example**

```r
rCopyEdit# Clean feedback text
feedback$text <- tolower(feedback$text)
feedback$text <- gsub("[[:punct:]]", "", feedback$text)
feedback <- feedback[!is.na(feedback$text) & feedback$text != "", ]
```

### 2. Sentiment Model Development

To automate labeling, I tested eight popular sentiment analysis approaches:

* **4 lexicon-based** methods (e.g., AFINN, NRC, Bing, SentiWordNet)
* **4 model-based** methods (pre-trained on tweets, books, movie reviews, etc.)

**Objective:**\
Identify the most accurate sentiment classification model for BYU’s survey feedback.

**Methodology:**

* Compared all models on a manually labeled validation set
* Used **accuracy** as the evaluation metric (balanced classes, equal cost of errors)
* Top two models reached between **80%–88% accuracy**
* Identified complementary error patterns between top models

**Key Insights:**

* Model 2 often caught errors missed by Model 1
* I created an ensemble model: trained a **random forest** to predict whether Model 1 would be wrong
* Final classifier: if Model 1 likely to miss → use Model 2; else → use Model 1
* Final ensemble achieved **\~95% accuracy**

### 3. Topic Analysis

To enrich the sentiment filter, I applied topic modeling techniques to uncover key themes across thousands of comments.

**Objective:**\
Make qualitative feedback more interpretable by grouping comments into meaningful topics.

**Methodology:**

* Applied **Latent Dirichlet Allocation (LDA)** to identify recurring themes (e.g., instructors, scheduling, curriculum)
* Cleaned and tokenized text to optimize topic coherence
* Validated topics by reviewing top keywords and representative comments

**Key Insights:**

* Topic modeling revealed patterns not easily detected through manual review
* Helped distinguish between programs struggling with logistics vs. content-related issues
* Enabled tracking of strengths and weaknesses by theme over time

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

### 4. Interactive Dashboard

To ensure usability, I packaged the models and insights into an intuitive, interactive dashboard for business units.

**Objective:**\
Empower non-technical stakeholders to explore and act on feedback insights.

**Methodology:**

* Built an **interactive Shiny dashboard** with filters for sentiment, program, demographic group, and topic
* Added visualizations: sentiment distribution charts, topic word clouds, and comment viewers
* Designed with responsiveness and usability in mind for regular departmental use

**Key Insights:**

* Departments could quickly locate negative feedback spikes by program and demographic
* Strengths from high-performing programs became visible and sharable
* The tool transformed open-text feedback into a practical resource for decision-making

***

## 📈 Results

* Created a high-accuracy (95%) feedback classification system
* Reduced feedback analysis time to **¼ of the original time**
* Provided business departments with a **filterable dashboard** to track strengths and pain points
* Empowered more data-driven decisions in program improvement and resource allocation

***

## 🛤️ Future Improvements

* **Monitor model drift** and revalidate model accuracy over time
* **Add interaction effects** (e.g., how sentiment varies by program-demographic combinations)
* Expand topic modeling to include **emerging trends** or unknown issues
* Introduce **confidence scores** for model predictions to aid user interpretation
* Build in **longitudinal tracking** to monitor how sentiment evolves over semesters

***

This project taught me how to blend natural language processing, model evaluation, and stakeholder needs into a scalable, impactful solution. Looking forward to applying these tools to more feedback-rich environments.

Thanks for reading, and keep the learning going!
