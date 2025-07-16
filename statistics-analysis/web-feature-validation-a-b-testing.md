# Web Feature Validation - A/B Testing

## 🛠️ Overview

As a Project Manager on a web development team, I led a series of A/B tests aimed at improving user experience and engagement on our website. By testing different design elements and feature rollouts—such as navigation menus and color themes—we were able to make data-informed decisions that enhanced usability and user experience. Each test was carefully designed, implemented, and statistically analyzed to ensure that changes were truly effective.

Jump right into my A/B Testing tool

[#id-5.-automate-the-process-using-googlesheet](web-feature-validation-a-b-testing.md#id-5.-automate-the-process-using-googlesheet "mention")

***

## 🔥 Problem Statement

How can we determine whether user preferences from a survey or A/B test reflect a real difference, rather than random chance?

**Example use case:**\
A design team proposes changing the website’s theme color from Blue to Green. They want to know:

> _Do users significantly prefer Green over Blue?_

A simple majority isn’t enough — we need statistical evidence to make a data-backed decision.

***

## 🧩 Project Pipeline

### **1. Formulate Hypotheses**

We used a one-tailed hypothesis test for proportions because our goal was to verify whether a majority of users preferred Green (i.e., more than 50%).

Let p represent the true proportion of users who prefer Green.

* Null Hypothesis (H₀): p ≤ 0.5\
  &#xNAN;_&#x55;sers are equally likely to prefer Green or Blue, or prefer Blue._
* Alternative Hypothesis (H₁): p > 0.5\
  &#xNAN;_&#x4D;ore than half of users prefer Green._

### 2. Collect and Prepare the Data

To capture user preference between the Green and Blue themes, we deployed a **pop-up survey** on a college website.

* The pop-up appeared **before users accessed content** or **as they were about to exit**.
*   Users were shown two screenshots (Green vs. Blue theme) and asked:

    > _"Which website theme do you prefer?"_
* They selected their preferred design with a single click.

**Survey Stats:**

* **Approximate site visitors during the period:** 2,500
* **Number of responses:** 100
* **Response rate:** 4%

This is consistent with typical **2–5% response rates** seen on higher ed websites for brief pop-up surveys.

**Prepared Data:**

* **Users who selected Green:** 71
* **Sample proportion:**
  * p\_hat = 0.71

### 3. Choose the Test and Run the Statistics

Since we were comparing a single observed proportion against a known benchmark (50%), we used a **one-sample z-test for proportions** with a confience level of 0.05.

The z-test statistic is given by:

<p align="center"><img src="../.gitbook/assets/Screen Shot 2025-06-28 at 10.29.54 PM.png" alt=""></p>

Substituting the values:

<p align="center"><img src="../.gitbook/assets/Screen Shot 2025-06-28 at 10.28.24 PM.png" alt=""></p>

Using a standard normal distribution table, a z-score of 4.2 corresponds to a **p-value ≈ 0.000013**.

### 4. Make the Inference

Since the p-value is **much lower than 0.05**, we **reject the null hypothesis**.

> **Conclusion:** There is strong statistical evidence that more than half of users prefer the **Green theme**. This validates the design team’s intuition and supports moving forward with this visual direction.

### 5.  Automate the Process using GoogleSheet

To automate the process for future A/B test I created a quick tool in Google Sheets were you input the numbers you get from the survey and get if your results where statistacally significant or not.

{% embed url="https://docs.google.com/spreadsheets/d/1IbDSvNQnrAf4NlyiGevS-75dXJEmCWzFH3g7dirWoo0/edit?usp=sharing" %}
A/B Testing Tool
{% endembed %}

***

### 📈 Conclusion

* Increased feature adoption and user engagement across several site areas
* Empowered stakeholders with **clear, evidence-backed recommendations** on design and content direction

***

### 🛤️ Future Improvements

* Automate A/B test setup and result analysis with python using Streamlit
* Explore **multi-variate testing** for more complex layout experiments

***

This project helped me bridge product thinking with statistical rigor, and proved that data-driven decisions lead to smarter, more user-centric designs.

Thanks for reading, and keep the learning going.
