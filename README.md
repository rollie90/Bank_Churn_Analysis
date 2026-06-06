# Bank_Churn_Analysis
Data analysis project to identify VIP customer churn and the capital impact on the banking portfolio.

**Project Quick Links:**

**Interactive Dashboard:** 
[View on Tableau Public](https://public.tableau.com/shared/ZMBQK4GKP?:display_count=n&:origin=viz_share_link)

**Validation Code:** 
[Open in Google Colab](https://colab.research.google.com/drive/1q25pubLK0XNCJLplrq2NI-l0vHk-Bqne#scrollTo=EQgGVZZ-DYv9&uniqifier=5)

**Executive Presentation:** 
[View Slides on Google Slides](https://docs.google.com/presentation/d/1JKlhcTYKxoBv5ohJJYY4zZHJxJUR0qZZYgx6UAENo1o/edit?slide=id.g3e9ea28c28f_2_15#slide=id.g3e9ea28c28f_2_15)

**Data Source**
The analysis is based on a public dataset from a financial institution available on Kaggle:

**Original Dataset:** 
[Bank Customer Churn Prediction (Kaggle)](https://www.kaggle.com/datasets/gauravtopre/bank-customer-churn-dataset?resource=download)

**Dataset Characteristics:** 
Contains records of 10,000 customers with demographic variables (age, country, gender) and financial variables (account balances, number of active products, estimated salary, and churn status).

Executive summary:
The fictional ABC Multinational Bank is facing a harsh situation where a big chunk of their customers is leaving the bank. In order to face the situation, the bank is requesting a data analysis project that will assist in identifying the key factors that the bank can use to predict customer churn by analyzing data from account holders. The analysis found that the biggest chunk of customer churn was represented mainly by a specific population group within the bank: Clients 40 years and older who own more than 4 bank products.

Technical Analysis and Methodology:
The first step was to upload the database to Google Colab and confirm whether the data was ready to be manipulated and analyzed.
Loading the data:

[Open in Google Colab] (https://colab.research.google.com/drive/1q25pubLK0XNCJLplrq2NI-l0vHk-Bqne#scrollTo=EQgGVZZ-DYv9&uniqifier=5)

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

I also played around with Tableau using different age groups and found out that when considering the ages 40+ the churn overall rate increased to 50% plus in Germany.


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

4- Finally, I wanted to calculate the overall impact of the churn for the bank in Germany to measure the real urgency of the matter. I utilized Python to calculate the financial burden of the churn rate for the bank in Germany, which resulted on a 32.61% of capital drain that amounts to $97,973,915.53, and verified my previous findings. I also used Tableau to calculate the churn rate, which turned out to be 32.44%. Finally, I created a dataviz for each.
Python code:
<details>
<summary>💻 Haz clic aquí para ver el código completo de validación</summary>
    '''Python
bank_germany = active_balance_df[active_balance_df['country']=='Germany']
churned_germany = churned_customers[churned_customers['country']=='Germany']
retained_germany = retained_customers[retained_customers['country']=='Germany']

total_bank_capital_germany = bank_germany['balance'].sum()
churn_germany_balance = churned_germany['balance'].sum()
retained_germany_balance = retained_germany['balance'].sum()
drain_percentage_germany = (churn_germany_balance/total_bank_capital_germany)*100
avg_churn_germany = churned_germany['balance'].mean()
avg_retained_germany = retained_germany['balance'].mean()
relative_change_germany = ((avg_churn_germany-avg_retained_germany)/avg_retained_germany)*100

print(f"Total Capital in Germany: ${total_bank_capital_germany:,.2f}")
print(f"Total Balance of Exited Customers in Germany: ${churn_germany_balance:,.2f}")
print(f"Total Balance of Retained Customers in Germany: ${retained_germany_balance:,.2f}")
print(f"Total Bank Capital Drain Percentage in Germany: {drain_percentage_germany:,.2f}%")
print(f"Average Balance of Exited Customers in Germany: ${avg_churn_germany:,.2f}")
print(f"Average Balance of Retained Customers in Germany: ${avg_retained_germany:,.2f}")
print(f"Value Differential (Churn vs. Retained) in Germany: {relative_change_germany:,.2f}%")
'''
</details>

Heatmap dataviz for balance drain in Germany:

<img width="524" height="634" alt="Balance Drain Germany" src="https://github.com/user-attachments/assets/f337a19a-ecb0-45b3-a27f-6a17130e7e55" />

Pie chart dataviz for churn rate in Germany:

<img width="1324" height="1044" alt="Churn Rate Germany" src="https://github.com/user-attachments/assets/a760e2cc-babc-428c-b69c-1d0489d41afc" />

Conclusion and Recommendations:

[View Slides on Google Slides](https://docs.google.com/presentation/d/1JKlhcTYKxoBv5ohJJYY4zZHJxJUR0qZZYgx6UAENo1o/edit?slide=id.g3e9ea28c28f_2_15#slide=id.g3e9ea28c28f_2_15)

Overall, the data indicates that Germany is the lead country with the highest churn rate of over 32%. Furthermore, regarding different factors and considerations, the research revails that the most critical group to churn are people aged 40 years plus who own 3 or more bank products with a churn rate of higher than 95%. Visualized as follows:

Pie chart:

<img width="519" height="630" alt="Pie Chart Drain Capital 40 plus" src="https://github.com/user-attachments/assets/e7bd25b1-934e-4c0a-a688-ea9ef71a4b70" />

Bar chart:

<img width="363" height="323" alt="Bar chart 3 products pls" src="https://github.com/user-attachments/assets/dd89c75e-75a9-4562-946f-aa0e70ac3395" />

Suggestions:

1- Qualitative analysis research on the targeted population with digital surveys and in person client profiling. The objective being to identidy core churn reasons: 
* Customer service satisfaction level.
* Score for bank performance and overall products & services vis à vis the competition.

2- Create a marketing loyalty campaign which targets the weak points:
* Product catalogue renovation.
* Promotions for the targeted population.
* Incentive programs.








