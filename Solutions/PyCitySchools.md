

```python
import pandas as pd
import os
```


```python
file_to_load_1 = os.path.join("../PyCitySchools/raw_data/schools_complete.csv")
file_to_load_2 = os.path.join("../PyCitySchools/raw_data/students_complete.csv")
```


```python
schools_df = pd.read_csv(file_to_load_1)
students_df = pd.read_csv(file_to_load_2)
```


```python
# rename 'name' column in schools_df
schools_df = schools_df.rename(columns={"name":"school"})
```


```python
schools_df.head()
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
      <th>School ID</th>
      <th>school</th>
      <th>type</th>
      <th>size</th>
      <th>budget</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>Huang High School</td>
      <td>District</td>
      <td>2917</td>
      <td>1910635</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>Figueroa High School</td>
      <td>District</td>
      <td>2949</td>
      <td>1884411</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>Shelton High School</td>
      <td>Charter</td>
      <td>1761</td>
      <td>1056600</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>Hernandez High School</td>
      <td>District</td>
      <td>4635</td>
      <td>3022020</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>Griffin High School</td>
      <td>Charter</td>
      <td>1468</td>
      <td>917500</td>
    </tr>
  </tbody>
</table>
</div>



# District Summary


```python
# district summary statistics
total_schools = len(schools_df['school'].unique())
# ^^^I can also use schools_df['name'].count(), but len(unique) method safer in case of duplicates. any better way?
total_students = schools_df['size'].sum()
total_budget = schools_df['budget'].sum()

# testing scores
# math
district_average_math = students_df['math_score'].mean()
count_passing_math = len(students_df.loc[students_df['math_score'] > 69])
percent_passing_math = (count_passing_math/total_students) * 100

# english
district_average_reading = students_df['reading_score'].mean()
count_passing_reading = len(students_df.loc[students_df['reading_score'] > 69])
percent_passing_reading = (count_passing_reading/total_students) * 100

# overall passing rate
percent_passing_overall = (percent_passing_math + percent_passing_reading) / 2

# translate above stats into dataframe
district_summary = pd.DataFrame({
    "School Count":total_schools, "Total District Students":total_students,
    "Total District Budget":total_budget, "Average Math Score": district_average_math,
    "Percent Passing Math":percent_passing_math, "Average Reading Score": district_average_reading,
    "Percent Passing Reading":[percent_passing_reading], "Overall Passing Rate":percent_passing_overall
})

# munge it baby
district_summary = round(district_summary, 2)
district_summary['Average Math Score'] = district_summary['Average Math Score'].map("{:.2f}%".format)
district_summary['Average Reading Score'] = district_summary['Average Reading Score'].map("{:.2f}%".format)
district_summary['Overall Passing Rate'] = district_summary['Overall Passing Rate'].map("{:.2f}%".format)
district_summary['Percent Passing Math'] = district_summary['Percent Passing Math'].map("{:.2f}%".format)
district_summary['Percent Passing Reading'] = district_summary['Percent Passing Reading'].map("{:.2f}%".format)
district_summary['Total District Budget'] = district_summary['Total District Budget'].map("${:.2f}".format)

district_summary
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
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>Overall Passing Rate</th>
      <th>Percent Passing Math</th>
      <th>Percent Passing Reading</th>
      <th>School Count</th>
      <th>Total District Budget</th>
      <th>Total District Students</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>78.99%</td>
      <td>81.88%</td>
      <td>80.39%</td>
      <td>74.98%</td>
      <td>85.81%</td>
      <td>15</td>
      <td>$24649428.00</td>
      <td>39170</td>
    </tr>
  </tbody>
</table>
</div>



# School Summary


```python
# combine both dataframes
combined_df = pd.merge(schools_df, students_df, how='inner', on='school')

# get school type
school_types = schools_df.set_index(['school'])['type']

# student count by school
school_student_count = combined_df['school'].value_counts()

# per school stats using groupbys
# budget by school, student per school
school_total_budget = combined_df.groupby('school').mean()['budget']
school_budget_per_student = (school_total_budget/school_student_count)

# average scores
school_average_math = combined_df.groupby('school').mean()['math_score']
school_average_reading = combined_df.groupby('school').mean()['reading_score']

# total passing, percent passing
# math
school_count_passing_math = combined_df.loc[combined_df['math_score'] > 69]
school_percent_passing_math = (school_count_passing_math.groupby('school').count()['name']/school_student_count) * 100

# english
school_count_passing_reading = combined_df.loc[combined_df['reading_score'] > 69]
school_percent_passing_reading = (school_count_passing_reading.groupby('school').count()['name']/school_student_count) * 100

# overall
school_overall_passing = ((school_percent_passing_math + school_percent_passing_reading) / 2)

# dump summary stats into dataframe
school_summary = pd.DataFrame({
    "Student Count":school_student_count, "Total Budget":school_total_budget,
    "Spending Per Student":school_budget_per_student, "Average Math Score":school_average_math,
    "Average Reading Score":school_average_reading, "Percent Passing Math":school_percent_passing_math,
    "Percent Passing Reading":school_percent_passing_reading, "Overall Passing Rate":school_overall_passing,
    "School Type":school_types
})

# munge school summary dataframe
school_summary = round(school_summary, 2)
# school_summary['Total Budget'] = school_summary['Total Budget'].map("${:,.2f}".format)
# school_summary['Spending Per Student'] = school_summary['Spending Per Student'].map("${:,.0f}".format)
# school_summary['Average Math Score'] = school_summary['Average Math Score'].map("{:,.2f}%".format)
# school_summary['Average Reading Score'] = school_summary['Average Reading Score'].map("{:,.2f}%".format)
# school_summary['Overall Passing Rate'] = school_summary['Overall Passing Rate'].map("{:,.2f}%".format)
# school_summary['Percent Passing Math'] = school_summary['Percent Passing Math'].map("{:,.2f}%".format)
# school_summary['Percent Passing Reading'] = school_summary['Percent Passing Reading'].map("{:,.2f}%".format)

school_summary
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
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>Overall Passing Rate</th>
      <th>Percent Passing Math</th>
      <th>Percent Passing Reading</th>
      <th>School Type</th>
      <th>Spending Per Student</th>
      <th>Student Count</th>
      <th>Total Budget</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>77.05</td>
      <td>81.03</td>
      <td>74.31</td>
      <td>66.68</td>
      <td>81.93</td>
      <td>District</td>
      <td>628.0</td>
      <td>4976</td>
      <td>3124928.0</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.06</td>
      <td>83.98</td>
      <td>95.59</td>
      <td>94.13</td>
      <td>97.04</td>
      <td>Charter</td>
      <td>582.0</td>
      <td>1858</td>
      <td>1081356.0</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.71</td>
      <td>81.16</td>
      <td>73.36</td>
      <td>65.99</td>
      <td>80.74</td>
      <td>District</td>
      <td>639.0</td>
      <td>2949</td>
      <td>1884411.0</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.10</td>
      <td>80.75</td>
      <td>73.80</td>
      <td>68.31</td>
      <td>79.30</td>
      <td>District</td>
      <td>644.0</td>
      <td>2739</td>
      <td>1763916.0</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.35</td>
      <td>83.82</td>
      <td>95.27</td>
      <td>93.39</td>
      <td>97.14</td>
      <td>Charter</td>
      <td>625.0</td>
      <td>1468</td>
      <td>917500.0</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.29</td>
      <td>80.93</td>
      <td>73.81</td>
      <td>66.75</td>
      <td>80.86</td>
      <td>District</td>
      <td>652.0</td>
      <td>4635</td>
      <td>3022020.0</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.80</td>
      <td>83.81</td>
      <td>94.38</td>
      <td>92.51</td>
      <td>96.25</td>
      <td>Charter</td>
      <td>581.0</td>
      <td>427</td>
      <td>248087.0</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>76.63</td>
      <td>81.18</td>
      <td>73.50</td>
      <td>65.68</td>
      <td>81.32</td>
      <td>District</td>
      <td>655.0</td>
      <td>2917</td>
      <td>1910635.0</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>77.07</td>
      <td>80.97</td>
      <td>73.64</td>
      <td>66.06</td>
      <td>81.22</td>
      <td>District</td>
      <td>650.0</td>
      <td>4761</td>
      <td>3094650.0</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.84</td>
      <td>84.04</td>
      <td>95.27</td>
      <td>94.59</td>
      <td>95.95</td>
      <td>Charter</td>
      <td>609.0</td>
      <td>962</td>
      <td>585858.0</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.84</td>
      <td>80.74</td>
      <td>73.29</td>
      <td>66.37</td>
      <td>80.22</td>
      <td>District</td>
      <td>637.0</td>
      <td>3999</td>
      <td>2547363.0</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.36</td>
      <td>83.73</td>
      <td>94.86</td>
      <td>93.87</td>
      <td>95.85</td>
      <td>Charter</td>
      <td>600.0</td>
      <td>1761</td>
      <td>1056600.0</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.42</td>
      <td>83.85</td>
      <td>95.29</td>
      <td>93.27</td>
      <td>97.31</td>
      <td>Charter</td>
      <td>638.0</td>
      <td>1635</td>
      <td>1043130.0</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.27</td>
      <td>83.99</td>
      <td>95.20</td>
      <td>93.87</td>
      <td>96.54</td>
      <td>Charter</td>
      <td>578.0</td>
      <td>2283</td>
      <td>1319574.0</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.68</td>
      <td>83.96</td>
      <td>94.97</td>
      <td>93.33</td>
      <td>96.61</td>
      <td>Charter</td>
      <td>583.0</td>
      <td>1800</td>
      <td>1049400.0</td>
    </tr>
  </tbody>
</table>
</div>



# Top Performing Schools by Passing Rate


```python
top_schools_passing = school_summary.sort_values('Overall Passing Rate', ascending=False)
top_schools_passing = top_schools_passing[:5]

# munge school summary dataframe
top_schools_passing = round(top_schools_passing, 2)
top_schools_passing['Total Budget'] = top_schools_passing['Total Budget'].map("${:,.2f}".format)
top_schools_passing['Spending Per Student'] = top_schools_passing['Spending Per Student'].map("${:,.0f}".format)
top_schools_passing['Average Math Score'] = top_schools_passing['Average Math Score'].map("{:,.2f}%".format)
top_schools_passing['Average Reading Score'] = top_schools_passing['Average Reading Score'].map("{:,.2f}%".format)
top_schools_passing['Overall Passing Rate'] = top_schools_passing['Overall Passing Rate'].map("{:,.2f}%".format)
top_schools_passing['Percent Passing Math'] = top_schools_passing['Percent Passing Math'].map("{:,.2f}%".format)
top_schools_passing['Percent Passing Reading'] = top_schools_passing['Percent Passing Reading'].map("{:,.2f}%".format)

top_schools_passing
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
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>Overall Passing Rate</th>
      <th>Percent Passing Math</th>
      <th>Percent Passing Reading</th>
      <th>School Type</th>
      <th>Spending Per Student</th>
      <th>Student Count</th>
      <th>Total Budget</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Cabrera High School</th>
      <td>83.06%</td>
      <td>83.98%</td>
      <td>95.59%</td>
      <td>94.13%</td>
      <td>97.04%</td>
      <td>Charter</td>
      <td>$582</td>
      <td>1858</td>
      <td>$1,081,356.00</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.42%</td>
      <td>83.85%</td>
      <td>95.29%</td>
      <td>93.27%</td>
      <td>97.31%</td>
      <td>Charter</td>
      <td>$638</td>
      <td>1635</td>
      <td>$1,043,130.00</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.35%</td>
      <td>83.82%</td>
      <td>95.27%</td>
      <td>93.39%</td>
      <td>97.14%</td>
      <td>Charter</td>
      <td>$625</td>
      <td>1468</td>
      <td>$917,500.00</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.84%</td>
      <td>84.04%</td>
      <td>95.27%</td>
      <td>94.59%</td>
      <td>95.95%</td>
      <td>Charter</td>
      <td>$609</td>
      <td>962</td>
      <td>$585,858.00</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.27%</td>
      <td>83.99%</td>
      <td>95.20%</td>
      <td>93.87%</td>
      <td>96.54%</td>
      <td>Charter</td>
      <td>$578</td>
      <td>2283</td>
      <td>$1,319,574.00</td>
    </tr>
  </tbody>
</table>
</div>



# Worst Performing Schools by Passing Rate


```python
worst_schools_passing = school_summary.sort_values('Overall Passing Rate')
worst_schools_passing = worst_schools_passing[:5]

# munge school summary dataframe
worst_schools_passing = round(worst_schools_passing, 2)
worst_schools_passing['Total Budget'] = worst_schools_passing['Total Budget'].map("${:,.2f}".format)
worst_schools_passing['Spending Per Student'] = worst_schools_passing['Spending Per Student'].map("${:,.0f}".format)
worst_schools_passing['Average Math Score'] = worst_schools_passing['Average Math Score'].map("{:,.2f}%".format)
worst_schools_passing['Average Reading Score'] = worst_schools_passing['Average Reading Score'].map("{:,.2f}%".format)
worst_schools_passing['Overall Passing Rate'] = worst_schools_passing['Overall Passing Rate'].map("{:,.2f}%".format)
worst_schools_passing['Percent Passing Math'] = worst_schools_passing['Percent Passing Math'].map("{:,.2f}%".format)
worst_schools_passing['Percent Passing Reading'] = worst_schools_passing['Percent Passing Reading'].map("{:,.2f}%".format)

worst_schools_passing.head()
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
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>Overall Passing Rate</th>
      <th>Percent Passing Math</th>
      <th>Percent Passing Reading</th>
      <th>School Type</th>
      <th>Spending Per Student</th>
      <th>Student Count</th>
      <th>Total Budget</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.84%</td>
      <td>80.74%</td>
      <td>73.29%</td>
      <td>66.37%</td>
      <td>80.22%</td>
      <td>District</td>
      <td>$637</td>
      <td>3999</td>
      <td>$2,547,363.00</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.71%</td>
      <td>81.16%</td>
      <td>73.36%</td>
      <td>65.99%</td>
      <td>80.74%</td>
      <td>District</td>
      <td>$639</td>
      <td>2949</td>
      <td>$1,884,411.00</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>76.63%</td>
      <td>81.18%</td>
      <td>73.50%</td>
      <td>65.68%</td>
      <td>81.32%</td>
      <td>District</td>
      <td>$655</td>
      <td>2917</td>
      <td>$1,910,635.00</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>77.07%</td>
      <td>80.97%</td>
      <td>73.64%</td>
      <td>66.06%</td>
      <td>81.22%</td>
      <td>District</td>
      <td>$650</td>
      <td>4761</td>
      <td>$3,094,650.00</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.10%</td>
      <td>80.75%</td>
      <td>73.80%</td>
      <td>68.31%</td>
      <td>79.30%</td>
      <td>District</td>
      <td>$644</td>
      <td>2739</td>
      <td>$1,763,916.00</td>
    </tr>
  </tbody>
</table>
</div>



# District Test Scores by Grade


```python
# create group by grade
grade_group = students_df.groupby('grade')

# math scores by grade
grades_math = grade_group['math_score'].mean()

# reading scores by grade
grades_reading = grade_group['reading_score'].mean()

# dump analysis in data frame
grade_score_summary = pd.DataFrame({
    "Average Math Score":grades_math,
    "Average Reading Score":grades_reading
})

# munge it
grade_score_summary = round(grade_score_summary, 2)
grade_score_summary['Average Math Score'] = grade_score_summary['Average Math Score'].map("{:,.2f}%".format)
grade_score_summary['Average Reading Score'] = grade_score_summary['Average Reading Score'].map("{:,.2f}%".format)

grade_score_summary
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
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
    </tr>
    <tr>
      <th>grade</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>10th</th>
      <td>78.94%</td>
      <td>81.87%</td>
    </tr>
    <tr>
      <th>11th</th>
      <td>79.08%</td>
      <td>81.89%</td>
    </tr>
    <tr>
      <th>12th</th>
      <td>78.99%</td>
      <td>81.82%</td>
    </tr>
    <tr>
      <th>9th</th>
      <td>78.94%</td>
      <td>81.91%</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Spending


```python
# split schools by average spending on student
bins = [570, 590, 610, 700]

# bin labels
group_labels = ['Low', 'Medium', 'High']

# add bins to dataframe
school_summary['Average Spending Per Student'] = pd.cut(school_summary['Spending Per Student'], bins, labels=group_labels)

# group by bins
spending_group = school_summary.groupby('Average Spending Per Student')
```


```python
# analysis by spending bin
spending_math_average = spending_group['Average Math Score'].mean()
spending_reading_average = spending_group['Average Reading Score'].mean()
spending_percent_math = spending_group['Percent Passing Math'].mean()
spending_percent_reading = spending_group['Percent Passing Reading'].mean()
spending_overall_passing = spending_group['Overall Passing Rate'].mean()

# dump analysis in table
spending_scores_summary = pd.DataFrame({
    "Average Math Score":spending_math_average, "Average Reading Score":spending_reading_average,
    "Percent Passing Math":spending_percent_math, "Percent Passing Reading":spending_percent_reading,
    "Overall Passing Rate":spending_overall_passing
})

# munge it
spending_scores_summary = round(spending_scores_summary, 2)
spending_scores_summary['Average Math Score'] = spending_scores_summary['Average Math Score'].map("{:,.2f}%".format)
spending_scores_summary['Average Reading Score'] = spending_scores_summary['Average Reading Score'].map("{:,.2f}%".format)
spending_scores_summary['Overall Passing Rate'] = spending_scores_summary['Overall Passing Rate'].map("{:,.2f}%".format)
spending_scores_summary['Percent Passing Reading'] = spending_scores_summary['Percent Passing Reading'].map("{:,.2f}%".format)
spending_scores_summary['Percent Passing Math'] = spending_scores_summary['Percent Passing Math'].map("{:,.2f}%".format)

spending_scores_summary
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
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>Overall Passing Rate</th>
      <th>Percent Passing Math</th>
      <th>Percent Passing Reading</th>
    </tr>
    <tr>
      <th>Average Spending Per Student</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Low</th>
      <td>83.45%</td>
      <td>83.94%</td>
      <td>95.04%</td>
      <td>93.46%</td>
      <td>96.61%</td>
    </tr>
    <tr>
      <th>Medium</th>
      <td>83.60%</td>
      <td>83.88%</td>
      <td>95.06%</td>
      <td>94.23%</td>
      <td>95.90%</td>
    </tr>
    <tr>
      <th>High</th>
      <td>78.38%</td>
      <td>81.60%</td>
      <td>78.47%</td>
      <td>72.50%</td>
      <td>84.45%</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Size


```python
# split schools by school size
bins = [0, 1500, 3000, 7000]

# bin labels
group_labels = ['Small', 'Medium', 'Large']

# add bins to dataframe
school_summary['School Size'] = pd.cut(school_summary['Student Count'], bins, labels=group_labels)

# group by bins
size_group = school_summary.groupby('School Size')
```


```python
# analysis by size bin
size_math_average = size_group['Average Math Score'].mean()
size_reading_average = size_group['Average Reading Score'].mean()
size_percent_math = size_group['Percent Passing Math'].mean()
size_percent_reading = size_group['Percent Passing Reading'].mean()
size_overall_passing = size_group['Overall Passing Rate'].mean()

# dump analysis in dataframe
size_scores_summary = pd.DataFrame({
    "Average Math Score":size_math_average, "Average Reading Score":size_reading_average,
    "Percent Passing Math":size_percent_math, "Percent Passing Reading":size_percent_reading,
    "Overall Passing Rate":size_overall_passing
})

# munge it
size_scores_summary = round(size_scores_summary, 2)
size_scores_summary['Average Math Score'] = size_scores_summary['Average Math Score'].map("{:,.2f}%".format)
size_scores_summary['Average Reading Score'] = size_scores_summary['Average Reading Score'].map("{:,.2f}%".format)
size_scores_summary['Overall Passing Rate'] = size_scores_summary['Overall Passing Rate'].map("{:,.2f}%".format)
size_scores_summary['Percent Passing Reading'] = size_scores_summary['Percent Passing Reading'].map("{:,.2f}%".format)
size_scores_summary['Percent Passing Math'] = size_scores_summary['Percent Passing Math'].map("{:,.2f}%".format)

size_scores_summary
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
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>Overall Passing Rate</th>
      <th>Percent Passing Math</th>
      <th>Percent Passing Reading</th>
    </tr>
    <tr>
      <th>School Size</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Small</th>
      <td>83.66%</td>
      <td>83.89%</td>
      <td>94.97%</td>
      <td>93.50%</td>
      <td>96.45%</td>
    </tr>
    <tr>
      <th>Medium</th>
      <td>80.90%</td>
      <td>82.82%</td>
      <td>87.07%</td>
      <td>83.56%</td>
      <td>90.59%</td>
    </tr>
    <tr>
      <th>Large</th>
      <td>77.06%</td>
      <td>80.92%</td>
      <td>73.76%</td>
      <td>66.46%</td>
      <td>81.06%</td>
    </tr>
  </tbody>
</table>
</div>



# Scores by School Type


```python
# group schools by type
type_groups = school_summary.groupby('School Type')

# testing scores by type
type_math_average = type_groups['Average Math Score'].mean()
type_reading_average = type_groups['Average Reading Score'].mean()
type_percent_math = type_groups['Percent Passing Math'].mean()
type_percent_reading = type_groups['Percent Passing Reading'].mean()
type_overall_passing = type_groups['Overall Passing Rate'].mean()

# type scores summary table
type_scores_summary = pd.DataFrame({
    "Average Math Score":type_math_average, "Average Reading Score":type_reading_average,
    "Percent Passing Math":type_percent_math, "Percent Passing Reading":type_percent_reading,
    "Overall Passing Rate":type_overall_passing
})

# munge it
type_scores_summary = round(type_scores_summary, 2)
type_scores_summary['Average Math Score'] = type_scores_summary['Average Math Score'].map("{:,.2f}%".format)
type_scores_summary['Average Reading Score'] = type_scores_summary['Average Reading Score'].map("{:,.2f}%".format)
type_scores_summary['Percent Passing Math'] = type_scores_summary['Percent Passing Math'].map("{:,.2f}%".format)
type_scores_summary['Percent Passing Reading'] = type_scores_summary['Percent Passing Reading'].map("{:,.2f}%".format)
type_scores_summary['Overall Passing Rate'] = type_scores_summary['Overall Passing Rate'].map("{:,.2f}%".format)

type_scores_summary
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
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>Overall Passing Rate</th>
      <th>Percent Passing Math</th>
      <th>Percent Passing Reading</th>
    </tr>
    <tr>
      <th>School Type</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Charter</th>
      <td>83.47%</td>
      <td>83.90%</td>
      <td>95.10%</td>
      <td>93.62%</td>
      <td>96.59%</td>
    </tr>
    <tr>
      <th>District</th>
      <td>76.96%</td>
      <td>80.97%</td>
      <td>73.67%</td>
      <td>66.55%</td>
      <td>80.80%</td>
    </tr>
  </tbody>
</table>
</div>


