# pandas-challenge
-------------


## Option 1: Heroes of Pymoli

![](Images/fantasy.png)


```python
import pandas as pd
import os
import numpy as np

df = pd.read_csv(os.path.join("Resources","purchase_data.csv"))
```

---------
### 1. Player Count
* Total Number of Players

<details><summary>code</summary>

```python
f"Total number of players: {len(set(df['SN']))}"
```

</details>    
    
`Output => 'Total number of players: 576'`

---------
### 2. Purchasing Analysis (Total)
* Number of Unique Items
* Average Purchase Price
* Total Number of Purchases
* Total Revenue

<details><summary>code</summary>

```python
pd.DataFrame({
    "Number of Unique Items": [len(set(df['Item ID']))],
    "Average Purchase Price" : [df['Price'].mean()],
    "Total Number of Purchases" : [len(set(df['Purchase ID']))],
    "Total Revenue" : df['Price'].sum()
}).style.format({"Average Purchase Price" : "${:.2f}", "Total Revenue": "${:,.2f}"})
```

</details>    
    
![](Images/image000002.png)

------------
### 3. Gender Demographics
* Percentage and Count of Male Players
* Percentage and Count of Female Players
* Percentage and Count of Other / Non-Disclosed

<details><summary>code</summary>

```python
male_players_count = len(set(df.groupby(by="Gender").get_group("Male")['SN']))
female_players_count = len(set(df.groupby(by="Gender").get_group("Female")['SN']))
other_count = len(set(df.groupby(by="Gender").get_group("Other / Non-Disclosed")['SN']))
total_players = male_players_count + female_players_count + other_count

def perc(x):
    return x/total_players*100

data = [male_players_count,female_players_count,other_count]

pd.DataFrame({
    "Count": data,
    "Percentage" : [perc(x) for x in data]
}, index=["Male","Female","Other / Non-Disclosed"]
).style.format({"Percentage": "{:.1f}%"})
```

</details>

![](Images/image000005.png)


-------------
### 4. Purchasing Analysis (Gender)
- The below each broken by gender
    - Purchase Count
    - Average Purchase Price
    - Total Purchase Value
    - Average Purchase Total per Person by Gender

<details><summary>code</summary>
    
```python
pd.DataFrame({
    "Purchase Count" : df.groupby(by="Gender")['Purchase ID'].count(),
    "Average Purchase Price" : df.groupby(by="Gender")['Price'].mean(),
    "Total Purchase Value" : df.groupby(by="Gender")['Price'].sum(),
    "Average Purchase Total per Person by Gender" : df.groupby(by="Gender")['Price'].sum()/np.array([female_players_count,male_players_count,other_count])
}).style.format({'Average Purchase Price': "${:.2f}",'Total Purchase Value': "${:,.2f}",'Average Purchase Total per Person by Gender': "${:.2f}"})
``` 

</details>

![](Images/image000006.png)

---------
### 5. Age Demographics
* The below each broken into bins of 4 years (i.e. <10, 10-14, 15-19, etc.)
    * Purchase Count
    * Average Purchase Price
    * Total Purchase Value
    * Average Purchase Total per Person by Age Group

<details><summary>code</summary>

```python
bins = [0] + [x*5 for x in range(2,10)] + [100]
df["Age Groups"] = pd.cut(df["Age"],bins, right = False, labels=["<10","10-14","15-19","20-24","25-29","30-34","35-39","40-44",">45"])
bin_group = df.groupby(by="Age Groups")
pd.DataFrame({
    "Purchase Count" : bin_group['Purchase ID'].size(),
    "Average Purchase Price" : bin_group['Price'].mean(),
    "Total Purchase Value" : bin_group['Price'].sum(),
    "Average Purchase Total per Person by Age Group" : bin_group['Price'].sum()/[len(set(bin_group['SN'].get_group(key))) for key in bin_group['SN'].groups.keys()]
}).style.format({'Average Purchase Price':"${:.2f}",'Total Purchase Value':"${:,.2f}","Average Purchase Total per Person by Age Group": "${:.2f}"})
```
    
</details>

![](Images/image000007.png)

-------------
### 6. Top Spenders
* Identify the the top 5 spenders in the game by total purchase value, then list (in a table):
    * SN
    * Purchase Count
    * Average Purchase Price
    * Total Purchase Value

<details><summary>code</summary>

```python
sn_group = df.groupby(by="SN")
top5 = sn_group['Price'].sum().nlargest(5).index
top5

df_top5 = pd.DataFrame(
    [],
    columns = ["SN","Purchase Count","Average Purchase Price","Total Purchase Value"]
)
df_top5

for index, user in enumerate(top5):
    df_top5.loc[index+1,"SN"] = user
    df_top5.loc[index+1,"Purchase Count"] = sn_group.get_group(user)["Purchase ID"].count()
    df_top5.loc[index+1,"Average Purchase Price"] = sn_group.get_group(user)["Price"].mean()
    df_top5.loc[index+1,"Total Purchase Value"] = sn_group.get_group(user)["Price"].sum()
    
df_top5.style.format({"Average Purchase Price":"${:.2f}","Total Purchase Value":"${:.2f}"})
```

</details>

![](Images/image000008.png)

--------
### 7. Most Popular Items
* Identify the 5 most popular items by purchase count, then list (in a table):
    * Item ID
    * Item Name
    * Purchase Count
    * Item Price
    * Total Purchase Value

<details><summary>code</summary>

```python
item_group = df.groupby(by="Item ID")
top5_items = df.groupby(by="Item ID").count().sort_values(by="Purchase ID",ascending=False).head().reset_index()['Item ID']

df_top5_items = pd.DataFrame(
    data = [],
    columns = ['Item ID','Item Name','Purchase Count','Item Price','Total Purchase Value']
)  

for index, item in enumerate(top5_items):
    df_top5_items.loc[index+1, 'Item ID'] = item
    df_top5_items.loc[index+1, 'Item Name'] = item_group.get_group(item).reset_index().loc[0,'Item Name'] 
    df_top5_items.loc[index+1, 'Purchase Count'] = item_group.get_group(item).count()[0]
    df_top5_items.loc[index+1, 'Item Price'] = ", ".join([f"${x}" for x in item_group.get_group(item).reset_index()['Price'].unique()])
    df_top5_items.loc[index+1, 'Total Purchase Value'] = item_group.get_group(item).reset_index()['Price'].sum()
    
df_top5_items.style.format({"Total Purchase Value": "${:.2f}"})
```                                                          

</details>    

![](Images/image000009.png)

---------
### 8. Most Profitable Items
* Sort the above table by total purchase value in descending order
* Optional: give the displayed data cleaner formatting
* Display a preview of the data frame

<details><summary>code</summary>

```python
df_profit = df.groupby(by="Item ID").sum()['Price'].reset_index().rename(columns={"Price":"Total Purchase Value"}).sort_values(by="Total Purchase Value",ascending=False).set_index("Item ID")

df_profit = df_profit.merge(df[["Item ID","Item Name"]].drop_duplicates(), how='inner', on="Item ID")
df_profit = df_profit.merge(item_group.count().rename(columns={'SN':'Purchase Count'})[['Purchase Count']], how="left", on="Item ID")

df_profit["Item Price"] = [''] * df_profit['Item ID'].size

for index, item in enumerate(df_profit['Item ID']):
    df_profit.loc[index,"Item Price"] = ", ".join(f"${x:.2f}" for x in item_group.get_group(item).reset_index()['Price'].unique())

df_profit.head(10).style.format({"Total Purchase Value": "${:.2f}"})
```
    
</details>

![](Images/image000010.png)

----------
### Final Considerations
##### Three (3) observable trends based on the data.
    1. Males make up the majority of the target market
    2. Age 20-24 has the most purchases as well as makes up the most gross profit.  Only one purchaser over age 45.
    3. Even the the top spenders didn't spend more than $20.