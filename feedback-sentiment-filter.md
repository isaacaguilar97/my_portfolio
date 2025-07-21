# Feedback Sentiment Filter

## 🛠️ Overview

Open-ended feedback from student satisfaction surveys often goes unread by business departments due to the overwhelming volume and time commitment required. To bridge this gap, I developed an **interactive feedback sentiment filter**—a tool that uses machine learning to automatically classify comment sentiment and organize it by program, respondent demographics, and key topics. The goal was to make qualitative feedback actionable, enabling faster, more targeted analysis to inform program improvements.

> _Note: Code snippets serve only as visual aids to represent the code used in the project. The actual code generated in this project is not shared for intellectual property reasons._

***

## 🔥 Problem Statement

How can we reduce the time it takes to analyze feedback comments in a more accessible, meaningful, and actionable way without overwhelming department staff?&#x20;

***

## 🧩 Project Pipeline

### 1. Data Collection & Preprocessing

I began by aggregating five years of survey responses from BYU Continuing Education. These surveys contained both structured data (e.g., program type, role, demographics) and unstructured text responses.

**Sources:**

* Qualtrics survey data including:
  * Program particpants and parents responses
  * Demographic metadata (role, age, program, etc.)

**Cleaning:**

* Removed duplicates and entries lacking satisfaction scores
* Filtered for entries with at least 80% completion
* Ensured reliable survey responses (avg. time spent was 1.6 to 2 minutes)

**Text Preprocessing:**

* Convert all text to lowercase to ensure uniformity
* Remove punctuation
* Split the text into individual words or tokens
* Remove common words (like “the”, “is”, “and”) that do not add much meaning
* Reduce words to their base form (Lemmatization)

> **Python Example**

```python
# Clean feedback text
fimport pandas as pd
import re
import nltk
from nltk.tokenize import word_tokenize
from nltk.corpus import stopwords, wordnet
from nltk.stem import WordNetLemmatizer

# Download required resources (run once)
nltk.download('punkt')
nltk.download('stopwords')
nltk.download('wordnet')
nltk.download('omw-1.4')

# Sample DataFrame
df = pd.DataFrame({
    'Comments': [
        "I absolutely loved the product!!!",
        "This isn't what I expected. Very disappointing.",
        "Service was good, but delivery was late.",
        "Would definitely recommend to my friends :)"
    ]
})

# Initialize tools
stop_words = set(stopwords.words('english'))
lemmatizer = WordNetLemmatizer()

# Define preprocessing function
def preprocess(text):
    # Lowercase
    text = text.lower()
    
    # Remove punctuation
    text = re.sub(r'[^\w\s]', '', text)
    
    # Tokenize
    tokens = word_tokenize(text)
    
    # Remove stopwords
    tokens = [word for word in tokens if word not in stop_words]
    
    # Lemmatize
    tokens = [lemmatizer.lemmatize(word) for word in tokens]
    
    return tokens

# Apply preprocessing to the Comments column
df['Processed_Comments'] = df['Comments'].apply(preprocess)

# Show result
print(df[['Comments', 'Processed_Comments']])

```

> Example of Tokenized Text

| Comments                                        | Processed\_Comments                             |
| ----------------------------------------------- | ----------------------------------------------- |
| I absolutely loved the product!!!               | \['absolutely', 'love', 'product']              |
| This isn't what I expected. Very disappointing. | \['expect', 'disappoint']                       |
| Service was good, but delivery was late.        | \['service', 'good', 'delivery', 'late']        |
| Would definitely recommend to my friends :)     | \['would', 'definitely', 'recommend', 'friend'] |

### 2. Model Selection

To automate labeling, I tested eight popular sentiment analysis approaches:

* 4 lexicon-based methods (e.g., sentimentr, syuzhet, TextBlob, VADER (NLTK))
  * Consisted of mapping words/tokens with sentiments (positive, neutral, or negative), counting the number of words with a given sentiment, and labeling the comment with the sentiment with the biggest count.
* 4 model-based methods (pre-trained on tweets, books, movie reviews, etc.)

**Objective:**

Identify the most accurate sentiment classification model for BYU’s survey feedback.

**Methodology:**

* Manualy have multiple team members label 600 comments with a given criteria
* Split 300 labels for training, 200 for validating, & 100 for testing
* Compared all models on labeled validation sets using
* Used **accuracy** as the evaluation metric (balanced classes, equal cost of errors)
* Identified complementary error patterns between top models

> RoBERTa Pre-trained Example

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification
from transformers import pipeline
import torch
import numpy as np

# Load the tokenizer and model
model_name = "cardiffnlp/twitter-roberta-base-sentiment"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSequenceClassification.from_pretrained(model_name)

# Sentiment labels for this model
labels = ['negative', 'neutral', 'positive']

# Define your sentiment pipeline
sentiment_pipeline = pipeline("sentiment-analysis", model=model, tokenizer=tokenizer)

# Example text
texts = [
    "I love this product! Works perfectly.",
    "It’s okay, but I expected more.",
    "Terrible experience. Will never buy again."
]

# Run sentiment analysis
for text in texts:
    result = sentiment_pipeline(text)[0]
    label = result['label']
    score = result['score']
    print(f"Text: {text}\n → Sentiment: {label} ({score:.2f})\n")
```

**Key Insights:**

* Top two models reached between **80%–88% accuracy**
* Top models were BERT based models (DistilBERT & RoBERTa)
* Model 2 often caught errors missed by Model 1

### 3. Model Improvement (Random Forest)

In order to take advantage of the errors that Model 2 was getting right from Model 1, I could somehow combine both models to improve the overall sentiment accuracy.

**Objective**

Develop a hybrid sentiment analysis system by training a Random Forest model to predict when **Model 1** (RoBERTa) is likely to misclassify a comment. Based on this prediction, dynamically switch to **Model 2** for those at-risk cases.

**Methodology:**

* Ran the **RoBERTa** model and collected its **output scores**, which represent the probability assigned to each sentiment class (e.g., `[0.05, 0.10, 0.85]` for negative, neutral, and positive).
* Split the dataset into a **subtraining set** and a **validation set**.
* Used the RoBERTa output scores as input features to train a **Random Forest model**.
* Created a binary target label indicating whether RoBERTa’s prediction was **correct or incorrect** for each comment.
* Trained the Random Forest and validated its performance, achieving **90% accuracy** in predicting when RoBERTa would fail.
* Wrote the final classification logic:
  * If the Random Forest predicts a likely error → **Use Model 2**
  * Otherwise → **Use Model 1**

> Simplified Example of Hybrid Model

```python
import numpy as np
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

# --- Simulated RoBERTa output scores (as features) ---
# Each row is: [negative_score, neutral_score, positive_score]
X_scores = np.array([
    [0.05, 0.10, 0.85],
    [0.40, 0.50, 0.10],
    [0.90, 0.05, 0.05],
    [0.20, 0.30, 0.50],
    [0.33, 0.33, 0.34]
])

# --- Ground truth labels ---
ground_truth = ['positive', 'neutral', 'negative', 'positive', 'neutral']

# --- RoBERTa predictions (argmax of scores) ---
roberta_preds = np.argmax(X_scores, axis=1)
label_map = {0: 'negative', 1: 'neutral', 2: 'positive'}
roberta_labels = [label_map[i] for i in roberta_preds]

# --- Target: was RoBERTa correct? (1 = correct, 0 = wrong) ---
y_target = np.array([int(p == g) for p, g in zip(roberta_labels, ground_truth)])

# --- Split into train/validation ---
X_train, X_val, y_train, y_val = train_test_split(X_scores, y_target, test_size=0.4, random_state=42)

# --- Train Random Forest to predict when RoBERTa is wrong ---
rf_model = RandomForestClassifier(random_state=42)
rf_model.fit(X_train, y_train)

# --- Predict on validation set ---
y_pred = rf_model.predict(X_val)
print("RF Accuracy (predicting RoBERTa's correctness):", accuracy_score(y_val, y_pred))

# --- Final ensemble logic ---
def hybrid_classifier(roberta_score):
    will_miss = rf_model.predict([roberta_score])[0] == 0
    if will_miss:
        return "Model 2 prediction"  # fallback
    else:
        return label_map[np.argmax(roberta_score)]  # use RoBERTa's label

# --- Test on new comment ---
new_score = [0.3, 0.4, 0.3] 
print("Final label:", hybrid_classifier(new_score))

```

**Key Insights:**

* The final **ensemble approach** achieved approximately **95% accuracy**.
* The new algorithm **improved performance by 7%** over the best single-model baseline.

### 4. Topic Analysis

To enrich the sentiment filter, I applied topic modeling techniques to uncover key themes across thousands of comments.

**Objective:**\
Make qualitative feedback more interpretable by grouping comments into meaningful topics.

**Methodology:**

* Cleaned and tokenized text to optimize topic coherence
* Applied **Latent Dirichlet Allocation (LDA)** to identify recurring themes
* Validated topics by reviewing top keywords and representative comments

**Key Insights:**

* Identified top reasons for enrollment by program
* Identified top areas of interest in participants (potentially new key indicators)
  * Examples include (teacher or program facilitator, content evaluation tool, customer service, content applicability in real life, etc.)

See topic modeling project for more information.

{% content-ref url="topic-modeling-cluster-analysis.md" %}
[topic-modeling-cluster-analysis.md](topic-modeling-cluster-analysis.md)
{% endcontent-ref %}

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
* Use recall instead of accruacy as my Random Forest performance metric, since I don't mind getting a few false negatives but it is crucial to get all of the true negatives (times when roBERTa model misslabed a comment) when validating the model.

***

This project taught me how to blend natural language processing, model evaluation, and stakeholder needs into a scalable, impactful solution. Looking forward to explore more on text analaysis.

Thanks for reading, and keep the learning going!
