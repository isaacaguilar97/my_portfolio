# Speed over Time - Simple Linear Regression

```python
import requests
import pandas as pd
from bs4 import BeautifulSoup
from sklearn.linear_model import LinearRegression
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np
```

    Matplotlib is building the font cache; this may take a moment.



```python
### Get my Table from the web into txt

# Send a GET request to the Wikipedia page
url = "https://en.wikipedia.org/wiki/Production_car_speed_record"
response = requests.get(url)

# Parse the HTML content using BeautifulSoup
soup = BeautifulSoup(response.content, "html.parser")

# Find the table containing the Production speed car world records
table = soup.find("table", {"class": "wikitable sortable"})

# Extract the headers of the table
headers = [header.text.strip() for header in table.find_all("th")]

# Extract the rows of the table
rows = []
for row in table.find_all("tr")[1:]:
    row_data = [data.text.strip() for data in row.find_all("td")]
    rows.append(row_data)

```


```python
### Turn my txt table into a pandas dataframe
df = pd.DataFrame(rows, columns=headers)
```


```python
df = df.drop(["Engine", "Comment"], axis=1)
```


```python
# Split these strings in Top Speed column and extract the numeric value in KMPH (Before we hit a space).
s = df["Top speed"].str.split().str[0]
df["Top speed"] = [f"{float(x):.3f}" for x in s.tolist()]
```


```python
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Year</th>
      <th>Make and model</th>
      <th>Top speed</th>
      <th>Number built</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1894</td>
      <td>Benz Velo</td>
      <td>20.000</td>
      <td>1,200</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1949</td>
      <td>Jaguar XK120</td>
      <td>200.500</td>
      <td>12,061[5]</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1955</td>
      <td>Mercedes-Benz 300 SL</td>
      <td>242.500</td>
      <td>1,400</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1959</td>
      <td>Aston Martin DB4 GT</td>
      <td>245.400</td>
      <td>75</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1960</td>
      <td>Aston Martin DB4 GT Zagato</td>
      <td>247.000</td>
      <td>19</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Make Year and Top Speed numeric variables
df['Top speed'] = pd.to_numeric(df['Top speed'], errors='coerce')
df['Year'] = pd.to_numeric(df['Year'])
```


```python
# Instantiate a LinearRegression model
model = LinearRegression()
x = df[["Year"]]
y = df["Top speed"]

# Fit the model using the Year and Top speed columns
my_model = model.fit(x, y)

```


```python
plt.scatter(x, y)

# Predict y values using the model and the x values for plotting the regression line
y_pred = my_model.predict(x)

# Plot the fitted line
plt.plot(x, y_pred, color='red')

# Set the axis labels and title
plt.xlabel('Year')
plt.ylabel('Top Speed')
plt.title('Top Speed vs Year')

# Show the plot
plt.show()

```


    
![png](Speed_Time_files/Speed_Time_8_0.png)
    



```python
R = my_model.score(x, y)
print(f"The coefficient of determination is:{R} Meaning {R*100} percent of the variation in Y can be attributed to X.")
```

    The coefficient of determination is:0.9796901374946986 Meaning 97.96901374946985 percent of the variation in Y can be attributed to X.



```python
# Compute the slope and intercept of the regression line
slope = model.coef_[0]
intercept = model.intercept_

# Print the result as a formatted string
print(f"The equation for the line of best fit is: Y = [{slope}] (X) - {intercept*-1}")
```

    The equation for the line of best fit is: Y = [3.4859072526475563] (X) - 6584.707048426586

