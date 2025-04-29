# High-Performing Product Recommender

## 🛠️ Overview

This project consisted on creating an automated system to **identify high-performing competitor products** that my old employer **did not offer at the time**, enabling **data-driven product expansion** recommendations. It integrated **regression modeling**, **feature extraction with CNNs**, **similarity search**, and an **interactive dashboard** built in **Shiny**.

## 🔥 Problem Statement

* Competitor catalogs contain **thousands of products** with varying levels of performance.
* Our company seeks to **expand smartly**, focusing only on **high-performing products** we **currently do not sell**.
* The project aims to **score competitor products**, **identify missing high-value items**, and **recommend** prioritized additions.

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

