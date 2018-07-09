

```python
import pandas as pd
import numpy as np
import os
```


```python
file_to_load = os.path.join("../HeroesofPymoli", "purchase_data2.json")
```


```python
df = pd.read_json(file_to_load, orient='records')
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
      <th>Age</th>
      <th>Gender</th>
      <th>Item ID</th>
      <th>Item Name</th>
      <th>Price</th>
      <th>SN</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>20</td>
      <td>Male</td>
      <td>93</td>
      <td>Apocalyptic Battlescythe</td>
      <td>4.49</td>
      <td>Iloni35</td>
    </tr>
    <tr>
      <th>1</th>
      <td>21</td>
      <td>Male</td>
      <td>12</td>
      <td>Dawne</td>
      <td>3.36</td>
      <td>Aidaira26</td>
    </tr>
    <tr>
      <th>2</th>
      <td>17</td>
      <td>Male</td>
      <td>5</td>
      <td>Putrid Fan</td>
      <td>2.63</td>
      <td>Irim47</td>
    </tr>
    <tr>
      <th>3</th>
      <td>17</td>
      <td>Male</td>
      <td>123</td>
      <td>Twilight's Carver</td>
      <td>2.55</td>
      <td>Irith83</td>
    </tr>
    <tr>
      <th>4</th>
      <td>22</td>
      <td>Male</td>
      <td>154</td>
      <td>Feral Katana</td>
      <td>4.11</td>
      <td>Philodil43</td>
    </tr>
  </tbody>
</table>
</div>




```python
# total player count
player_demographics = df.loc[:, ["Gender", "Age", "SN"]]
player_demographics = player_demographics.drop_duplicates()
player_count = player_demographics.count()[0]

pd.DataFrame({"Total Players":[player_count]})
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
      <th>Total Players</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>74</td>
    </tr>
  </tbody>
</table>
</div>



# Purchasing Analysis


```python
# unique item count
unique_item_count = len(df['Item ID'].unique())

# average purchase price
average_purchase_price = df['Price'].mean()

# total number of purchase
total_purchase_count = df.count()[0]

# total revenue
total_revenue = df['Price'].sum()

# convert to purchasing analysis dataframe
purchasing_summary = pd.DataFrame({"Total Item Count":[unique_item_count], "Average Purchase Price":[average_purchase_price],
                 "Total Purchase Count":[total_purchase_count], "Total Revenue":[total_revenue]})

# quick data munging
purchasing_summary = round(purchasing_summary, 2)
purchasing_summary["Average Purchase Price"] = purchasing_summary["Average Purchase Price"].map("${:,.2f}".format)
purchasing_summary["Total Revenue"] = purchasing_summary["Total Revenue"].map("${:,.2f}".format)

# # printed out, non-truncated data
# purchasing_analysis = (
#     f"---------------------------------"
#     f"\nTotal item count: {unique_item_count}"
#     f"\nAverage purchase price: {average_purchase_price}"
#     f"\nTotal purchase count: {total_purchase_count}"
#     f"\nTotal revenue: {total_revenue}"
#     f"\n---------------------------------"
# )

# print(purchasing_analysis)

# in dataframe format, truncated
purchasing_summary
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
      <th>Average Purchase Price</th>
      <th>Total Item Count</th>
      <th>Total Purchase Count</th>
      <th>Total Revenue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>$2.92</td>
      <td>64</td>
      <td>78</td>
      <td>$228.10</td>
    </tr>
  </tbody>
</table>
</div>



# Gender Demographics


```python
gender_demographics_totals = player_demographics["Gender"].value_counts()
gender_demographics_percents = gender_demographics_totals / player_count * 100
gender_summary = pd.DataFrame({"Total Count": gender_demographics_totals, "Percentage of Players": gender_demographics_percents})

# quick data munging
gender_summary = round(gender_summary, 2)
gender_summary['Percentage of Players'] = gender_summary['Percentage of Players'].map("{:,.2f}%".format)

gender_summary
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
      <th>Percentage of Players</th>
      <th>Total Count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Male</th>
      <td>81.08%</td>
      <td>60</td>
    </tr>
    <tr>
      <th>Female</th>
      <td>17.57%</td>
      <td>13</td>
    </tr>
    <tr>
      <th>Other / Non-Disclosed</th>
      <td>1.35%</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



# Purchasing Analysis by Gender


```python
# group df by gender
gender_group = df.groupby(df['Gender'])

# get stats by gender
gender_purchase_count = gender_group['SN'].count()
gender_average_purchase_price = gender_group['Price'].mean()
gender_total_purchase_value = gender_group['Price'].sum()
gender_normalized_totals = (gender_total_purchase_value / gender_purchase_count) * 100

# dump gender purchasing stats into summary dataframe
gender_purchase_summary = pd.DataFrame({"Purchase Count":gender_purchase_count,
                                      "Average Purchase Price":gender_average_purchase_price,
                                       "Total Purchase Value":gender_total_purchase_value,
                                       "Normalized Totals":gender_normalized_totals})

# quick data munging
gender_purchase_summary = round(gender_purchase_summary, 2)
gender_purchase_summary['Average Purchase Price'] = gender_purchase_summary['Average Purchase Price'].map("${:,.2f}".format)
gender_purchase_summary['Normalized Totals'] = gender_purchase_summary['Normalized Totals'].map("${:,.2f}".format)
gender_purchase_summary['Total Purchase Value'] = gender_purchase_summary['Total Purchase Value'].map("${:,.2f}".format)

gender_purchase_summary

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
      <th>Average Purchase Price</th>
      <th>Normalized Totals</th>
      <th>Purchase Count</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>Gender</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Female</th>
      <td>$3.18</td>
      <td>$318.31</td>
      <td>13</td>
      <td>$41.38</td>
    </tr>
    <tr>
      <th>Male</th>
      <td>$2.88</td>
      <td>$288.44</td>
      <td>64</td>
      <td>$184.60</td>
    </tr>
    <tr>
      <th>Other / Non-Disclosed</th>
      <td>$2.12</td>
      <td>$212.00</td>
      <td>1</td>
      <td>$2.12</td>
    </tr>
  </tbody>
</table>
</div>



# Age Demographics


```python
# split data into bins on age
bins = [0, 10, 18, 25, 100]

# groups for each bin
group_names = ['Child', 'Teenager', 'Young Adult', 'Adult']
```


```python
# add age group column to dataframe
df['Age Group'] = pd.cut(df['Age'], bins, labels=group_names)
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
      <th>Age</th>
      <th>Gender</th>
      <th>Item ID</th>
      <th>Item Name</th>
      <th>Price</th>
      <th>SN</th>
      <th>Age Group</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>20</td>
      <td>Male</td>
      <td>93</td>
      <td>Apocalyptic Battlescythe</td>
      <td>4.49</td>
      <td>Iloni35</td>
      <td>Young Adult</td>
    </tr>
    <tr>
      <th>1</th>
      <td>21</td>
      <td>Male</td>
      <td>12</td>
      <td>Dawne</td>
      <td>3.36</td>
      <td>Aidaira26</td>
      <td>Young Adult</td>
    </tr>
    <tr>
      <th>2</th>
      <td>17</td>
      <td>Male</td>
      <td>5</td>
      <td>Putrid Fan</td>
      <td>2.63</td>
      <td>Irim47</td>
      <td>Teenager</td>
    </tr>
    <tr>
      <th>3</th>
      <td>17</td>
      <td>Male</td>
      <td>123</td>
      <td>Twilight's Carver</td>
      <td>2.55</td>
      <td>Irith83</td>
      <td>Teenager</td>
    </tr>
    <tr>
      <th>4</th>
      <td>22</td>
      <td>Male</td>
      <td>154</td>
      <td>Feral Katana</td>
      <td>4.11</td>
      <td>Philodil43</td>
      <td>Young Adult</td>
    </tr>
  </tbody>
</table>
</div>




```python
# analysis by age group
age_group = df.groupby(df['Age Group'])

# gather stats by age group
age_group_purchase_count = age_group['Price'].count()
age_group_average_purchase_price = age_group['Price'].mean()
age_group_total_purchase_value = age_group['Price'].sum()
age_group_normalized_totals = (age_group_total_purchase_value/age_group_purchase_count) * 100

# build age group summary dataframe
age_group_summary = pd.DataFrame({
    "Purchase Count":age_group_purchase_count,
    "Average Purchase Price":age_group_average_purchase_price,
    "Total Purchase Value":age_group_total_purchase_value,
    "Normalized Totals":age_group_normalized_totals
})

# quick data mungin
age_group_summary = round(age_group_summary, 2)
age_group_summary['Average Purchase Price'] = age_group_summary['Average Purchase Price'].map("${:,.2f}".format)
age_group_summary['Normalized Totals'] = age_group_summary['Normalized Totals'].map("${:,.2f}".format)
age_group_summary['Total Purchase Value'] = age_group_summary['Total Purchase Value'].map("${:,.2f}".format)

age_group_summary
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
      <th>Average Purchase Price</th>
      <th>Normalized Totals</th>
      <th>Purchase Count</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>Age Group</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Child</th>
      <td>$2.76</td>
      <td>$276.40</td>
      <td>5</td>
      <td>$13.82</td>
    </tr>
    <tr>
      <th>Teenager</th>
      <td>$2.81</td>
      <td>$281.21</td>
      <td>14</td>
      <td>$39.37</td>
    </tr>
    <tr>
      <th>Young Adult</th>
      <td>$2.98</td>
      <td>$297.56</td>
      <td>43</td>
      <td>$127.95</td>
    </tr>
    <tr>
      <th>Adult</th>
      <td>$2.93</td>
      <td>$293.50</td>
      <td>16</td>
      <td>$46.96</td>
    </tr>
  </tbody>
</table>
</div>



# Top Spenders


```python
# group by player (SN)
spenders_group = df.groupby(['SN'])

# top spender statistics
spenders_sum = spenders_group['Price'].sum()
spenders_count = spenders_group['Price'].count()
spenders_average = spenders_group['Price'].mean()

# convert spender summary to dataframe
spender_summary = pd.DataFrame({
    "Total Spent":spenders_sum,
    "Purchase Count":spenders_count,
    "Average Purchase Price":spenders_average
})


# sort table by total spent, pull top 5 rows (spenders)
spender_summary = spender_summary.sort_values('Total Spent', ascending=False)
spender_summary = spender_summary[:5]

# quick data mungin
spender_summary = round(spender_summary, 2)
spender_summary['Average Purchase Price'] = spender_summary['Average Purchase Price'].map("${:,.2f}".format)
spender_summary['Total Spent'] = spender_summary['Total Spent'].map("${:,.2f}".format)
spender_summary
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
      <th>Average Purchase Price</th>
      <th>Purchase Count</th>
      <th>Total Spent</th>
    </tr>
    <tr>
      <th>SN</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Sundaky74</th>
      <td>$3.70</td>
      <td>2</td>
      <td>$7.41</td>
    </tr>
    <tr>
      <th>Aidaira26</th>
      <td>$2.56</td>
      <td>2</td>
      <td>$5.13</td>
    </tr>
    <tr>
      <th>Eusty71</th>
      <td>$4.81</td>
      <td>1</td>
      <td>$4.81</td>
    </tr>
    <tr>
      <th>Chanirra64</th>
      <td>$4.78</td>
      <td>1</td>
      <td>$4.78</td>
    </tr>
    <tr>
      <th>Alarap40</th>
      <td>$4.71</td>
      <td>1</td>
      <td>$4.71</td>
    </tr>
  </tbody>
</table>
</div>



# Most Popular Items


```python
# group by item
items_group = df.groupby(['Item ID', 'Item Name'])

# popular items statistics
purchase_count = items_group['Item Name'].count()
item_price = items_group['Price'].mean()
total_purchase_value = items_group['Price'].sum()

# convert items summary to dataframe
items_summary = pd.DataFrame({
    "Purchase Count": purchase_count,
    "Item Price": item_price,
    "Total Purchase Value": total_purchase_value
})

# sort table by purchase count, pull top 5 rows (items)
popular_summary = items_summary.sort_values('Purchase Count', ascending=False)
popular_summary = popular_summary[:5]

# data mungin
popular_summary['Item Price'] = popular_summary['Item Price'].map("${:,.2f}".format)
popular_summary['Total Purchase Value'] = popular_summary['Total Purchase Value'].map("${:,.2f}".format)


popular_summary
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
      <th></th>
      <th>Item Price</th>
      <th>Purchase Count</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>Item ID</th>
      <th>Item Name</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>94</th>
      <th>Mourning Blade</th>
      <td>$3.64</td>
      <td>3</td>
      <td>$10.92</td>
    </tr>
    <tr>
      <th>90</th>
      <th>Betrayer</th>
      <td>$4.12</td>
      <td>2</td>
      <td>$8.24</td>
    </tr>
    <tr>
      <th>111</th>
      <th>Misery's End</th>
      <td>$1.79</td>
      <td>2</td>
      <td>$3.58</td>
    </tr>
    <tr>
      <th>64</th>
      <th>Fusion Pummel</th>
      <td>$2.42</td>
      <td>2</td>
      <td>$4.84</td>
    </tr>
    <tr>
      <th>154</th>
      <th>Feral Katana</th>
      <td>$4.11</td>
      <td>2</td>
      <td>$8.22</td>
    </tr>
  </tbody>
</table>
</div>



# Most Profitable Items


```python
# this time, sort items_summary dataframe by total purchase value--not purchase count
profitable_items = items_summary.sort_values('Total Purchase Value', ascending=False)

# slice top 5 rows (top 5 items by total purchase value)
profitable_items = profitable_items[:5]

profitable_items
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
      <th></th>
      <th>Item Price</th>
      <th>Purchase Count</th>
      <th>Total Purchase Value</th>
    </tr>
    <tr>
      <th>Item ID</th>
      <th>Item Name</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>94</th>
      <th>Mourning Blade</th>
      <td>3.64</td>
      <td>3</td>
      <td>10.92</td>
    </tr>
    <tr>
      <th>117</th>
      <th>Heartstriker, Legacy of the Light</th>
      <td>4.71</td>
      <td>2</td>
      <td>9.42</td>
    </tr>
    <tr>
      <th>93</th>
      <th>Apocalyptic Battlescythe</th>
      <td>4.49</td>
      <td>2</td>
      <td>8.98</td>
    </tr>
    <tr>
      <th>90</th>
      <th>Betrayer</th>
      <td>4.12</td>
      <td>2</td>
      <td>8.24</td>
    </tr>
    <tr>
      <th>154</th>
      <th>Feral Katana</th>
      <td>4.11</td>
      <td>2</td>
      <td>8.22</td>
    </tr>
  </tbody>
</table>
</div>


