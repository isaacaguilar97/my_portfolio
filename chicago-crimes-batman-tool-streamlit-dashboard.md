---
description: >-
  Have you ever wondered how batman gets so quickly to catch the bad guys? Well,
  he has a tool like this.
---

# Chicago Crimes Batman Tool - Streamlit Dashboard

## 🛠️ Overview

This project centers on an interactive dashboard built to **predict violent crime rates** based on user-defined community characteristics. Instead of focusing on complex modeling details, this tool emphasizes **how people can interact with crime data directly** to simulate when is crime most likely to happen and where.

Created during a data competition, the goal was to move beyond static analysis and provide a **dynamic, hands-on experience** where users can adjust inputs like poverty, unemployment, or education levels, and immediately see how those changes affect violent crime predictions.

Try it out: [Violent Crime Prediction Dashboard](https://violentcrimes-mc2ay6qo6bz3rdzsjqn8nz.streamlit.app/)

## 🔥 Problem Statement

The central problem this dashboard addresses is:

* _How can we make complex crime prediction models usable and understandable by the average person — especially policy analysts or community leaders?_

This leads to a more interactive question:

* _What happens to predicted crime rates when certain social or economic conditions change?_

***

## 🧪 How the Dashboard Works

The dashboard is designed to help users **understand and simulate violent crime patterns** in Chicago’s District 11 — specifically, to explore how different weather, time, and socioeconomic factors influence the number of predicted violent crimes per hour.

Full code is available on Github but the dashboard itself is best experienced live

{% embed url="https://github.com/isaacaguilar97/violent_crimes/tree/main" %}

### **1. User Input: Adjust Key Conditions**

On the left sidebar, users can interactively modify over a dozen factors believed to influence crime, including:

* **Unemployment rate**
* **Temperature and humidity**
* **Wind speed, snow, and rainfall**
* **Visibility, radiation, cloud cover**
* **Holidays, full moons, day vs. night**

These values are based on real ranges from the data and can be adjusted to simulate different conditions.

<figure><img src=".gitbook/assets/Screen Shot 2025-07-17 at 8.40.31 AM.png" alt=""><figcaption></figcaption></figure>

Also, depending on the day of the the week and the time, we can predict where is crime most likely to happen.

<figure><img src=".gitbook/assets/Screen Shot 2025-07-17 at 8.55.58 AM.png" alt=""><figcaption></figcaption></figure>

### **2. Multiple Pre-trained Models Behind the Scenes**

Instead of using one generic model, this dashboard loads and selects **one of over 20 trained models** depending on:

* The **type of violent crime** selected (e.g., Robbery, Homicide, Intimidation, etc.)
* Or whether the user is viewing the **total number of crimes ("n\_crimes")** overall

Each model was trained separately and saved using `joblib`, then loaded dynamically when the dashboard starts. This allows for **customized prediction logic** for different crime types, increasing accuracy and contextual relevance.

```python
# Select model based on user input
if 'Primary Type_' + v_type + '_model' in model_name:
    selected_model = model
```

### **3. Preprocessing & Feature Engineering**

The user's input is automatically converted into a structured format that mirrors what the model expects. This includes:

* Creating **dummy variables** for categorical inputs like `month`, `hour`, and `day of the week`
* Adding binary indicators for **holiday**, **full moon**, and **day/night**
* Ensuring that **missing or unused columns are added with default values**, so the model input shape is always correct

```python
pythonCopyEdit# Fill missing dummy columns to ensure consistency with training
for col in expected_columns:
    if col not in user_input.columns:
        user_input[col] = 0
```

### **4. Real-Time Prediction**

Once the input is fully prepared, it is passed to the selected model to generate **hour-by-hour predictions** for a chosen week. These predictions are displayed visually so users can explore:

* When crime is most likely to occur
* How crime rates shift based on different scenarios

```python
# Predict number of violent crimes per hour
predictions = selected_model.predict(new_obs)
```

### **5. Focus on Interpretability**

Although the underlying models are built with machine learning, the dashboard emphasizes **interpretable inputs** (like weather and unemployment) and lets users see how changes in these inputs drive predicted changes in crime — without needing to understand the model architecture.

***

## 💡Key Takeaways

This project was one of the most rewarding experiences in my data journey so far. Beyond building models or analyzing data, I truly enjoyed the challenge of creating a **tool with real users in mind** — something interactive, informative, and potentially impactful. It taught me how to combine technical skills with design thinking to deliver insights that are **accessible and meaningful**. Most importantly, it reminded me that data tools can go beyond numbers — they can empower people, spark curiosity, and support smarter decisions in our communities.

Thanks for reading, and keep the learning going!
