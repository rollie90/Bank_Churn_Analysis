# Bank_Churn_Analysis
Banking and Customer Loyalty Study
Executive summary:
The fictional ABC Multinational Bank is facing a harsh situation where a big chunk of their customers is leaving the bank. In order to face the situation, the bank is requesting a data analysis project that will assist in identifying the key factors that the bank can use to predict customer churn by analyzing data from account holders. The analysis found that the biggest chunk of customer churn was represented mainly by a specific population group within the bank: Clients 40 years and older who own more than 4 bank products.

Technical Analysis and Methodology:
The first step was to upload the database to Google Colab and confirm whether the data was ready to be manipulated and analyzed.
Loading the data:
<details>
<summary>💻 Haz clic aquí para ver el código completo de validación</summary>
    '''python
import duckdb
import numpy as np
import pandas as pd

path = '/content/Bank Customer Churn Prediction.csv'
query = f"""
SELECT * FROM read_csv_auto('{path}')
"""
cla = duckdb.query(query).to_df() #For Churn Level Analysis
print(f"Dataset sucessfully loaded. Total registers: {len(cla)}")
print(cla.head())
'''
</details>

Verifying the data:
<details>
<summary>💻 Haz clic aquí para ver el código completo de validación</summary>
    '''python
cla.info()
cla.describe(include='all')
'''
</details>

Because the data was clean, with no nulls nor duplicates, I proceeded with the analysis.
1- The first thing that was noticiable was that the data showed that Germany had largest porcentage of churn compared to Spain and France with a 32% of churn. Here I used SQL to obtain the information and Tableau to visualize it with a heatmap:
<details>
<summary>💻 Haz clic aquí para ver el código completo de validación</summary>
    '''SQL
query_kpi = f"""
SELECT
    country,
    COUNT(*) as total_clients,
    SUM(churn) as total_churn,
    ROUND(AVG(churn) * 100, 2) as churn_percentage
FROM read_csv_auto('{path}')
GROUP BY country
ORDER BY churn_percentage DESC
"""

kpi_country = duckdb.query(query_kpi).to_df()  #Churn Percetage by country
print(kpi_country)
'''
</details>

<img width="510" height="567" alt="Country Heat map" src="https://github.com/user-attachments/assets/829783ba-55f3-4875-9b8a-ce573fd924a3" />

2- I used the library of matplotlib to create an histogram to verify if there was a specific age group we should focus on in Germany:
<details>
<summary>💻 Haz clic aquí para ver el código completo de validación</summary>
    '''python
import matplotlib.pyplot as plt
import seaborn as sns
import numpy as np
import pandas as pd

germany = cla[cla['country'] == 'Germany']

plt.figure(figsize=(10, 6))

sns.histplot(data=germany, x='age', hue='churn', bins=20, kde=True, palette='viridis')

plt.title('Churn Age Distribution in Germany')
plt.xlabel('Age')
plt.ylabel('Client Number')
plt.show()
'''
</details>

<img width="1067" height="690" alt="Histogram Age Group Ger" src="https://github.com/user-attachments/assets/9a126d00-2761-4400-8d59-a10ad15ade7b" />

I also played around with Tableau using different age groups and found out that when considering the ages 40+ the churn rate increased to over 50% and 50+ to 60% overall in Germany.
Age 40+:

<img width="521" height="579" alt="Age group 40+" src="https://github.com/user-attachments/assets/04b2a39e-451d-4814-81e5-0379a6fa05b8" />

Age 50+: 

<img width="516" height="582" alt="Age Group 50+" src="https://github.com/user-attachments/assets/435dce3c-5138-4d02-8462-cc7b562dfd71" />

3- Experimenting with the variables lead me to find a pattern with the quantity of bank products own by clients. The result was impressive:
Clients who own 4+ bank products have a churn rate of 100%. 
Code:
<details>
<summary>💻 Haz clic aquí para ver el código completo de validación</summary>
    '''SQL
query_kpi = f"""
SELECT
    country,
    products_number,
    ROUND(AVG(churn) * 100,2) as churn_percentage_germany by products
FROM read_csv_auto('{path}')
WHERE country = 'Germany'
GROUP BY country, products_number
ORDER BY churn_percentage_germany DESC
"""

kpi_germany_products = duckdb.query(query_kpi).to_df()  #Churn Percentage in Germany
print(kpi_germany_products)
'''
</details>

Visual:

<img width="1214" height="723" alt="Products_ Churn Rate overall" src="https://github.com/user-attachments/assets/aba78637-2521-40f0-a542-da5699d9e9f5" />

Furthermore, if considering the age groups 40+ from the previous assesment the churn rate increased to more than 90% for clients who own 3+ bank products. As visualized with Tableau:

<img width="374" height="312" alt="Products_Churn rate age 40+" src="https://github.com/user-attachments/assets/1f5ae7d7-890e-49ad-9208-a8fbd2cdc7c9" />

4- Finally, I wanted to calculate the overall impact of the churn in Germany for the bank


