# High-Performing Product Recommender

## 🛠️ Overview

This project consisted of creating an automated system to **identify high-performing competitor products** that the company **did not offer at the time**, enabling **data-driven product expansion** recommendations. It integrated **regression modeling**, **feature extraction with CNNs**, **similarity search**, and an **interactive dashboard** built in **Shiny**.

{% hint style="info" %}
Code snippets serve only as visual aids to represent the code used in the project. The actual code, graphs, and images generated in this project are not shared for intellectual property reasons.&#x20;
{% endhint %}

***

## 🔥 Problem Statement.

* The company sought to **expand smartly**, focusing on high-performing products it did not sell at the time.
* The project aimed to score competitor products, identify missing high-value items, and recommend prioritized additions.

***

## 🧩 Project Pipeline

### 1. Data Collection and Preprocessing

The first steps consisted of retrieving and organizing relevant data provided by a third-party vendor, containing details about the competitors' products.&#x20;

* **Sources:**\
  Data was sourced from an **Oracle database** containing records of competitor product transactions, including:
  * Product descriptions
  * Sales figures
  * Timestamps
  * Product images
* **Cleaning and Transformation:**\
  SQL was used to clean, transform, and prepare the data for analysis and modeling, including:
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
    sales_amount,
    qty,
    event_date_time,
    image_url,
    category,
    material,
    weight
FROM competitor_products
WHERE sales_amount IS NOT NULL
  AND sales_amount > 0;
```

### 2. Performance Score Creation (Regression Model)

Once there was a clean and consistent table with the data of interest, a way to identify high performing products had to be created.&#x20;

**Objective:**

Create a product score that allows to group products by performance (High, Regular, & Low), taking into consideration the effect of time in sales (hour, day of week, month, holiday, etc.).

**Methodology:**

* Create a new dataset with aggregated data that displays the average sales per hour.
* Build a Multiple Linear Regression model using the new dataset:
  * Independent Variables:\
    Hour, Day of Week, Week of Year, Month, Year, Holiday, Weekend
  * Dependent Variable:\
    Average Sales per hour (amount in dollars)
  * Measure how well the model captures the general trend of the data (Forecast Plot)
* Compute and map the corresponding predicted sales per hour to the original/transactional dataset.
* Calculate a Time Coefficient as the ratio of predicted sales to actual sales.
* Compute a transaction score:
  * _`Transaction_Score = Sales × TimeCoefficient`_
    * _`Sales`_ represent the average sales per transaction/observation.
    * `TimeCoefficient` is the effectiveness multiplier based on the regression model (whether that time boosts or hurts sales in general).
* Compute product score:
  * Aggregate the average transaction score per product&#x20;

This way products are rewarded when they have high sales at bad times, and they get penalized when they have low sales at favorable times.&#x20;

> Example (R):

```r
# Load necessary library
library(dplyr)

# Assume 'aggregated_data' is your data frame with the average sales per hour

# Fit the multiple linear regression
model <- lm(sales ~ hour + day_of_week + week_of_year + month + year + holiday + weekend, data = aggregated_data)

# Generate Forcast Plot to measure goodness of model to capture general trend of data
# ...

# Predict sales based on the time features
aggregated_data$predicted_sales <- predict(model, aggregated_data)

# Create the Time Coefficient as the ratio of predicted sales to actual sales
aggregated_data$time_coefficient <- aggregated_data$predicted_sales / aggregated_data$sales

# Now compute the Transaction Scores
aggregated_data$transaction_score <- (competitor_data$sales)  * (aggregated_data$time_coefficient)

# Get the Product Score by aggregating the transaction scores by product
product_scores <- aggregated_data %>%
  group_by(product_id) %>%
  summarise(product_score = mean(transaction_score))

# View the data
head(product_scores)
```

**High-Performing Product Definition:**

Based on the distribution of product scores, we labeled each product with a specific performance level.

* Only the **top 15%** of products (those scoring above 80) were considered **high performers**.
* The middle **70%** of products (those scoring between 50 to 80) were considered **regular performers**.
* The bottom **15%** of products (those scoring below 50) were considered **low performers**.

### 3. Feature Extraction (CNN on Images)

The next step in the process was figuring out if the company sales similar products to the competitor's high-performing items. To do so, both images and products features could help find possible matches, but images should be translated into something the computer is able to understand and make relationships with.&#x20;

**Objective:**

Capture visual characteristics of product images (colors, shapes, shades, objects, etc.) in a numeric format (vectors) to compare similarity between products.

**Methodology:**

* Convert images into vectors (numeric arrays representing the size and pixels of an image: width, height, color depth - intensity of Red, Green, & Blue per pixel).
* Input my image vectors into a pre-trained Convolutional Neural Network used to classify the company's product images.
* Extract the **image feature vectors** (visual characteristics of an image) by removing the final classification layer from the predicted output.

These image features identify the objects in each image, the spacing between them, the colors used, and many other characteristics that could be used to compare if two images represent the same object or idea.&#x20;

> Example (Scikit-Learn):

```python
import numpy as np
from sklearn.preprocessing import StandardScaler
from sklearn.neighbors import NearestNeighbors
# ----------------------------
# Data
# ----------------------------
# 3 products with 2048-d image features (simulated)
image_vectors = np.random.rand(3, 2048)
# Structured features: category (one-hot), material (one-hot), price, weight
# Example shape: [category_1, category_2, material_1, material_2, price, weight]
structured_features = np.array([
   [1, 0, 0, 1, 49.99, 0.8],
   [0, 1, 1, 0, 59.99, 0.6],
   [1, 0, 1, 0, 39.99, 0.9],
])
# New product to search against
new_image_vector = np.random.rand(1, 2048)
new_structured_features = np.array([[0, 1, 0, 1, 54.99, 0.75]])
# ----------------------------
# Preprocess structured features
# ----------------------------
# Scale price and weight
scaler = StandardScaler()
structured_scaled = scaler.fit_transform(structured_features)
new_structured_scaled = scaler.transform(new_structured_features)
# Apply weights (importance)
image_weight = 3
category_weight = 5
material_weight = 2
price_weight = 1
weight_weight = 1
# Split feature groups
category = structured_scaled[:, 0:2] * category_weight
material = structured_scaled[:, 2:4] * material_weight
price = structured_scaled[:, 4:5] * price_weight
weight_val = structured_scaled[:, 5:6] * weight_weight
combined_structured = np.hstack([category, material, price, weight_val])
# Same for the new product
new_cat = new_structured_scaled[:, 0:2] * category_weight
new_mat = new_structured_scaled[:, 2:4] * material_weight
new_price = new_structured_scaled[:, 4:5] * price_weight
new_weight = new_structured_scaled[:, 5:6] * weight_weight
new_structured_combined = np.hstack([new_cat, new_mat, new_price, new_weight])
# ----------------------------
# Combine all features
# ----------------------------
# Apply image vector weighting
image_vectors_weighted = image_vectors * image_weight
new_image_vector_weighted = new_image_vector * image_weight
# Final combined vectors
product_vectors = np.hstack([image_vectors_weighted, combined_structured])
new_product_vector = np.hstack([new_image_vector_weighted, new_structured_combined])
# ----------------------------
# Similarity Search (Euclidean)
# ----------------------------
knn = NearestNeighbors(n_neighbors=2, metric='euclidean')
knn.fit(product_vectors)
distances, indices = knn.kneighbors(new_product_vector)
# Output result
print("Closest products:")
print("Indices:", indices[0])
print("Distances:", distances[0])
```

### 4. Product Similarity Search (K-Nearest Neighbors)

With the images turned into something the computer can understand, a similarity model can be created with the image features and the product features.&#x20;

**Objective:**

Calculate a similarity score that compares the similarity/proximity between each of the competitor's high-performing products and the company's products.

**Methodology:**

* Collect and preprocess the company's products data on its own table the same way the competitor's that was preprocessed (including image feature extraction).
* Continue preprocessing data on both tables (competitor\_data, company\_data)
  * Scale numeric variables into standard scale (mean = 0, sd = 1)
  * One-hot encode categorical variables (create 1 column per category containing 0s and 1s)
  * Weight variables by feature importance (including image features vectors)
* Fit **K-Nearest Neighbors (KNN)** algorithm using the company's data
  * Turn image features and product features into one long vector per product to feed the model
  * Specify parameters: Use Euclidean distance & 3 nearest neighbors
* Find top 3 closest products per competitor's item
  * Apply model to each of the competitor's products
  * Select the 3 products with the lowest distance/score

By this point, there is a table with the competitor's products, their product features, and their 3 most similar items in the company.

### 5. Gap Detection

The challenge was now to create a cutoff to differentiate between matching products, and products that the company had opportunity to incorporate in its catalog.&#x20;

**Objective:**

Identify high-performing products without a sufficiently similar match in our catalog.

**Methodology:**

* Empirically identify similar products and their similarity scores.
* Set the minimum of those scores as the threshold.
* Products with **no match** within the distance threshold were **flagged as gaps**.
* Recommended these for potential addition.

From the 7K+ high-performing products, around 1.4K (20%) where marked as recommended additions to the company.

### 6. Interactive Recommendation Dashboard (Shiny)

The final task was to develop an interactive dashboard to support decision makers in identifying product opportunities for the company’s catalog. The dashboard enabled users to explore competitor products, filter them by performance level, and quickly assess whether similar items already exist in the company’s offerings. For each high-performing competitor product, it displayed the top three most similar products currently sold, along with their key features and representative images.

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

This tool provided a clear and efficient way to spot product gaps, evaluate alignment with market trends, and make informed decisions about potential catalog additions.

***

## 📈 Results

| Metric                              | Result                           |
| ----------------------------------- | -------------------------------- |
| Competitor Products Analyzed        | 50,000                           |
| High-Performing Products Identified | 7,000 (\~14%)                    |
| Recommended Additions               | 1,400 (\~20% of High Performers) |

Based on estimated average monthly sales of $2,000 per high-performing jewelry product and a typical cost of goods around $800, incorporating approximately 1,400 high-performing competitor items into the company’s catalog could generate an estimated $2.8 million in monthly revenue and over **$20 million in annual gross profit**.&#x20;

This highlights the significant profit potential of targeted product expansion based on similarity analysis and competitor benchmarking.

***

## 🛤️ Next Steps

1. Enhance Competitor's Product Scoring
   * Improve the product score formula by incorporating marginal profit per item (i.e., unit sale price minus cost of goods sold) to better reflect financial impact.
   * Extend the regression model by integrating additional structured product features (e.g., material, weight, category, brand) to improve the accuracy of sales predictions.
2. &#x20;Refine Similarity Scoring
   * &#x20;Develop two separate similarity scores: one based on structured product features and another based on image embeddings.
   * Combine both scores into a single composite metric using a weighted approach, giving more weight to structured attributes to prevent visual similarity from dominating the results.
3. Optimize Similarity Thresholds
   * Replace a fixed similarity cutoff with category-specific thresholds (e.g., different thresholds for rings, necklaces, bracelets) to better reflect the variability in product space.
   * Use a data-driven method such as the Elbow Method on distance distributions or Silhouette Analysis to quantitatively determine optimal thresholds for identifying product gaps.

***

Identifying gaps through data isn't just about what competitors offer, it's about recognizing where your catalog has room to grow with purpose and precision.&#x20;

Thank for reading and keep the learning going.
