# High-Performing Product Recommender

## 🛠️ Overview

This project consisted on creating an automated system to **identify high-performing competitor products** that my old employer **did not offer at the time**, enabling **data-driven product expansion** recommendations. It integrated **regression modeling**, **feature extraction with CNNs**, **similarity search**, and an **interactive dashboard** built in **Shiny**.

***

## 🔥 Problem Statement

* Competitor catalogs contain **thousands of products** with varying levels of performance.
* Our company seeks to **expand smartly**, focusing only on **high-performing products** we **currently do not sell**.
* The project aims to **score competitor products**, **identify missing high-value items**, and **recommend** prioritized additions.

***

## 🧩 Project Pipeline

### 1. Data Collection and Preprocessing

* **Sources:**\
  Data was sourced from an **Oracle database** containing competitor product catalogs, including:
  * Product descriptions
  * Pricing
  * Sales figures
  * Timestamps
  * Product images
* **Cleaning and Transformation:**\
  SQL was used to clean, transform, and prepare the data for modeling, inlcuding:
  * Handling missing values
  * Normalizing timestamps
  * Joining across multiple tables
  * Ensuring consistent feature naming and units
  * Removing duplicates and invalid entries

> Example (SQL):

```plsql
SELECT 
    product_id,
    product_name,
    price,
    sales_count,
    image_url,
    timestamp,
    category
FROM competitor_products
WHERE price IS NOT NULL
  AND sales_count > 0;
```

### 2. Performance Score Creation (Regression Model)

**Objective:**

Understand how **time of day** (controlling for day of week, month, holiday, etc.) affects **sales** and use this to enhance product scoring.

**Methodology:**

* Built a Multiple Linear Regression model:
  * Independent Variables:\
    Time of day, Day of Week, Week of Year, Month, Year, Holiday, Weekend
  * Dependent Variable:\
    Sales (amount in dollars)
* Extracted a time\_of\_day coefficient from the regression to adjust product sales based on sales timing.

**Product Score Formula:**

_`Product Score = (SalesVolume/QuantitySold) × (Time*TimeCoefficient/60)`_

* **SalesVolume / QuantitySold:**
  * The **average price per unit sold** (normalized revenue).
* **Time × TimeCoefficient:**
  * `Time` represents the **actual time.**
  * `TimeCoefficient` is the **effectiveness multiplier** based on the regression (whether that time boosts or hurts price).
* **Dividing by 60:**
  * Normalizes the `Time × TimeCoefficient` product to a comparable scale ("minutes" are being adjusted into a "per-hour" concept).

This way if a product is sold at a highly favorable time (according to the model) and had a strong price/unit, it will get a higher product score.

> Example (R):

```r
# Load necessary library
library(dplyr)

# Assume 'competitor_data' is your data frame

# Fit the multiple linear regression
model <- lm(price ~ minute + hour + day_of_week + week_of_year + month + year + holiday + weekend, data = competitor_data)

# Predict price based on the time features
competitor_data$predicted_price <- predict(model, competitor_data)

# Create the Time Coefficient as the ratio of predicted price to actual price
competitor_data$time_coefficient <- competitor_data$predicted_price / competitor_data$price

# Now compute the Product Score
competitor_data$product_score <- (competitor_data$sales_volume / competitor_data$quantity_sold) * (competitor_data$time_coefficient / 60)

# View the data
head(competitor_data)

```

**High-Performing Product Definition:**

* Only the **top 15%** of products (those scoring above 80) were considered **high performers**.

### 3. Feature Extraction (Images)

**Objective:**

Capture visual characteristics of products that structured metadata (like price or category) cannot express.

**Methodology:**

* Extracted **image feature vectors** using a pre-trained **ResNet50 CNN** (removing final classification layer).
* **Structured features** (e.g., price, category) and **image vectors** were treated **separately**.

**Why separate?**

* **Advantages:** Allows flexibility to weight structured data and visual data differently.
* **Tradeoff:** Combining them (by concatenating vectors) could allow a "unified" distance but might overweight one type.
* **Recommended:** Separate at first, but later experiment with **concatenation after normalization** if higher precision is required.

> Example (PyTorch):

```python
pythonCopyEditresnet50 = models.resnet50(pretrained=True)
feature_extractor = torch.nn.Sequential(*(list(resnet50.children())[:-1]))
# ... feature extraction code
```

### 4. Product Similarity Search (K-Nearest Neighbors)

**Objective:**

Identify whether a **similar product** already exists in our catalog.

**Methodology:**

* Modeled product similarity separately:
  * **Image vectors:** CNN embeddings
  * **Structured vectors:** Price, category, sales
* Applied **K-Nearest Neighbors (KNN)** algorithm with **Euclidean distance**.
* **Matching Criteria:**
  * Top 3 nearest neighbors were retrieved.
  * **Upper 20%** of matches (based on similarity scores) were considered a "good enough match".

> Threshold rationale:

* 20% cut-off ensures we focus on **closest matches** only.
* Balances between **recall** (finding enough candidates) and **precision** (quality of matches).
* A data-driven alternative: Analyze the distribution of distances and choose a cut-off at the elbow point (distance histogram inflection).

### 5. Gap Detection

**Objective:**

Identify high-performing products without a sufficiently similar match in our catalog.

**Methodology:**

* Products with **no match** within the distance threshold were **flagged as gaps**.
* Recommended these for potential addition.

### 6. Interactive Recommendation Dashboard (Shiny)

**Objective:**

Provide business users an **easy, interactive view** of product opportunities.

**Dashboard Features:**

* **Filters:**
  * Product performance (high, medium, low)
  * Product type (ring, necklace, bracelet, etc.)
  * Match status (Matched, No Match, All)
* **View Per Product:**
  * Product name
  * Product image
  * Product features
  * Top 3 closest matches with images and features
  * Match status (yes/no)

> Example (Shiny R):

```r
rCopyEditfluidPage(
  titlePanel("High Performing Product Gaps"),
  sidebarLayout(
    sidebarPanel(
      selectInput("performance", "Performance Level:", choices = c("All", "High", "Medium", "Low")),
      selectInput("type", "Product Type:", choices = unique(products$type)),
      checkboxInput("showNoMatch", "Show Only Products Without Match", value = FALSE)
    ),
    mainPanel(
      DT::dataTableOutput("recommendationTable")
    )
  )
)
```

***

## 📈 Results

| Metric                              | Result                           |
| ----------------------------------- | -------------------------------- |
| Competitor Products Analyzed        | 50,000                           |
| High-Performing Products Identified | 7,000 (\~14%)                    |
| Recommended Additions               | 1,400 (\~20% of High Performers) |

**Why 20%?**

* Business operations can realistically absorb **only a portion** of new products each cycle.
* The top 20% were chosen based on:
  * Highest predicted performance scores.
  * Availability of strong product images.
  * Feasibility to source or manufacture quickly.

***

## 🧠 Key Learnings

* Combining time-of-day adjustments into product scoring captured **hidden seasonality effects**.
* Separately treating image features and structured features provided **interpretable** similarity insights.
* An upper 20% similarity threshold provided **high-confidence matches** while avoiding false positives.
* Business users appreciated the ability to **explore gaps interactively**, leading to faster product approval.

***

## 🛤️ Next Steps

* Improve score formula by taking into consideration the marginal profit per item.
* Dynamic thresholding by category (jewelry vs. electronics vs. home goods).
* Feedback loop to capture accepted/rejected recommendations and retrain the similarity model.
