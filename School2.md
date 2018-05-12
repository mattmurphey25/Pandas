

```python
import pandas as pd
```


```python
csv_path_1 = "C:/Users/mattm/pandas-challenge/raw_data/schools_complete.csv"
csv_path_2 = "C:/Users/mattm/pandas-challenge/raw_data/students_complete.csv"

schools_df = pd.read_csv(csv_path_1, encoding="utf-8")
students_df = pd.read_csv(csv_path_2, encoding="utf-8")
schools_df = schools_df.rename(columns={'name': 'school'})
```


```python
school_merge = schools_df.merge(students_df, on = "school")
```


```python
total_schools = schools_df["school"].count()

total_students = schools_df["size"].sum()

total_budget = schools_df["budget"].sum()

average_math = school_merge["math_score"].mean()

average_reading = school_merge["reading_score"].mean()

amount_passing_math = school_merge[school_merge["math_score"] >= 70].count()["math_score"]
percent_passing_math = (100 * amount_passing_math) / total_students

amount_passing_reading = school_merge[school_merge["reading_score"] >= 70].count()["reading_score"]
percent_passing_reading = (100 * amount_passing_reading) / total_students

total_passing = amount_passing_math + amount_passing_reading
percent_passing_total = (percent_passing_math + percent_passing_reading) / 2
```


```python
district_data = {"Total Schools": total_schools, "Total Students": total_students, "Total Budget": total_budget, "Average Math Score": average_math, "Average Reading Score": average_reading, "% Passing Math": percent_passing_math, "% Passing Reading": percent_passing_reading, "Overall Passing Rate": percent_passing_total}
district_table = pd.DataFrame([district_data])
district_table = district_table[["Total Schools", "Total Students", "Total Budget", "Average Math Score", "Average Reading Score", "% Passing Math", "% Passing Reading", "Overall Passing Rate"]]
district_table["Total Students"] = district_table["Total Students"].map("{:,}".format)
district_table["Total Budget"] = district_table["Total Budget"].map("${:,.2f}".format)
district_table
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
      <th>Total Schools</th>
      <th>Total Students</th>
      <th>Total Budget</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>15</td>
      <td>39,170</td>
      <td>$24,649,428.00</td>
      <td>78.985371</td>
      <td>81.87784</td>
      <td>74.980853</td>
      <td>85.805463</td>
      <td>80.393158</td>
    </tr>
  </tbody>
</table>
</div>




```python
bins = [0, 69, 100]

passing = [0,1]

passing_math = pd.cut(school_merge["math_score"], bins, labels=passing)
passing_reading = pd.cut(school_merge["reading_score"], bins, labels=passing)
```


```python
school_merge["Passing Math"] = passing_math
school_merge["Passing Reading"] = passing_reading

school_merge["Passing Math"] = pd.to_numeric(school_merge["Passing Math"])
school_merge["Passing Reading"] = pd.to_numeric(school_merge["Passing Reading"])
school_merge["Total Passing"] = (school_merge["Passing Math"] + school_merge["Passing Reading"]) / 2
school_merge["Total Students"] = 1

school_merge["% Passing Math"] = 100 * school_merge["Passing Math"] / school_merge["size"]
school_merge["% Passing Reading"] = 100 * school_merge["Passing Reading"] / school_merge["size"]
school_merge["Overall Passing Rate"] = 100 * (school_merge["Passing Math"] + school_merge["Passing Reading"]) / (2 * school_merge["size"])
```


```python
school_merge_passing = school_merge[["school", "% Passing Math", "% Passing Reading", "Overall Passing Rate"]]
school_merge_passing_group = school_merge_passing.groupby(["school"])
school_merge_passing_group_sum = school_merge_passing_group.sum()
```


```python
total_by_school = schools_df[["school", "type", "size", "budget"]]
total_by_school_group = total_by_school.groupby(["school"])
total_comparison = total_by_school_group.sum()
total_comparison["Per Student Budget"] = total_comparison["budget"] / total_comparison["size"]
```


```python
total_comparison_2 = total_comparison.merge(school_merge_passing_group_sum, how='outer', left_index=True, right_index=True)
```


```python
scores = students_df[["school", "math_score", "reading_score"]]
scores_group = scores.groupby(["school"])
score_average = scores_group.mean()
```


```python
total_comparison_3 = total_comparison_2.merge(score_average, how='outer', left_index=True, right_index=True)
```


```python
school_type = schools_df[["school", "type"]]
school_type_2 = school_type.groupby(["school"])
school_type_3 = school_type_2.max()
school_type_3
total_comparison_4 = total_comparison_3.merge(school_type_3, how='outer', left_index=True, right_index=True)
```


```python
total_comparison_4.rename(columns={"size": "Total Students", "budget": "Total Student Budget", "math_score": "Average Math Score", "reading_score": "Average Reading Score", "type": "School Type"}, inplace=True)
total_comparison_4
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
      <th>Total Students</th>
      <th>Total Student Budget</th>
      <th>Per Student Budget</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>School Type</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>4976</td>
      <td>3124928</td>
      <td>628.0</td>
      <td>66.680064</td>
      <td>81.933280</td>
      <td>74.306672</td>
      <td>77.048432</td>
      <td>81.033963</td>
      <td>District</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>1858</td>
      <td>1081356</td>
      <td>582.0</td>
      <td>94.133477</td>
      <td>97.039828</td>
      <td>95.586652</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>Charter</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>2949</td>
      <td>1884411</td>
      <td>639.0</td>
      <td>65.988471</td>
      <td>80.739234</td>
      <td>73.363852</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>District</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>2739</td>
      <td>1763916</td>
      <td>644.0</td>
      <td>68.309602</td>
      <td>79.299014</td>
      <td>73.804308</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>District</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>1468</td>
      <td>917500</td>
      <td>625.0</td>
      <td>93.392371</td>
      <td>97.138965</td>
      <td>95.265668</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>Charter</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>4635</td>
      <td>3022020</td>
      <td>652.0</td>
      <td>66.752967</td>
      <td>80.862999</td>
      <td>73.807983</td>
      <td>77.289752</td>
      <td>80.934412</td>
      <td>District</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>427</td>
      <td>248087</td>
      <td>581.0</td>
      <td>92.505855</td>
      <td>96.252927</td>
      <td>94.379391</td>
      <td>83.803279</td>
      <td>83.814988</td>
      <td>Charter</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>2917</td>
      <td>1910635</td>
      <td>655.0</td>
      <td>65.683922</td>
      <td>81.316421</td>
      <td>73.500171</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>District</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>4761</td>
      <td>3094650</td>
      <td>650.0</td>
      <td>66.057551</td>
      <td>81.222432</td>
      <td>73.639992</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>District</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>962</td>
      <td>585858</td>
      <td>609.0</td>
      <td>94.594595</td>
      <td>95.945946</td>
      <td>95.270270</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>Charter</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>3999</td>
      <td>2547363</td>
      <td>637.0</td>
      <td>66.366592</td>
      <td>80.220055</td>
      <td>73.293323</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>District</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>1761</td>
      <td>1056600</td>
      <td>600.0</td>
      <td>93.867121</td>
      <td>95.854628</td>
      <td>94.860875</td>
      <td>83.359455</td>
      <td>83.725724</td>
      <td>Charter</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>1635</td>
      <td>1043130</td>
      <td>638.0</td>
      <td>93.272171</td>
      <td>97.308869</td>
      <td>95.290520</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>Charter</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>2283</td>
      <td>1319574</td>
      <td>578.0</td>
      <td>93.867718</td>
      <td>96.539641</td>
      <td>95.203679</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>Charter</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>1800</td>
      <td>1049400</td>
      <td>583.0</td>
      <td>93.333333</td>
      <td>96.611111</td>
      <td>94.972222</td>
      <td>83.682222</td>
      <td>83.955000</td>
      <td>Charter</td>
    </tr>
  </tbody>
</table>
</div>




```python
best_school_score = total_comparison_4.sort_values(by="Overall Passing Rate", ascending = False)
best_school_score.head(5)
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
      <th>Total Students</th>
      <th>Total Student Budget</th>
      <th>Per Student Budget</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>School Type</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Cabrera High School</th>
      <td>1858</td>
      <td>1081356</td>
      <td>582.0</td>
      <td>94.133477</td>
      <td>97.039828</td>
      <td>95.586652</td>
      <td>83.061895</td>
      <td>83.975780</td>
      <td>Charter</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>1635</td>
      <td>1043130</td>
      <td>638.0</td>
      <td>93.272171</td>
      <td>97.308869</td>
      <td>95.290520</td>
      <td>83.418349</td>
      <td>83.848930</td>
      <td>Charter</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>962</td>
      <td>585858</td>
      <td>609.0</td>
      <td>94.594595</td>
      <td>95.945946</td>
      <td>95.270270</td>
      <td>83.839917</td>
      <td>84.044699</td>
      <td>Charter</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>1468</td>
      <td>917500</td>
      <td>625.0</td>
      <td>93.392371</td>
      <td>97.138965</td>
      <td>95.265668</td>
      <td>83.351499</td>
      <td>83.816757</td>
      <td>Charter</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>2283</td>
      <td>1319574</td>
      <td>578.0</td>
      <td>93.867718</td>
      <td>96.539641</td>
      <td>95.203679</td>
      <td>83.274201</td>
      <td>83.989488</td>
      <td>Charter</td>
    </tr>
  </tbody>
</table>
</div>




```python
best_school_score.tail(5)
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
      <th>Total Students</th>
      <th>Total Student Budget</th>
      <th>Per Student Budget</th>
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
      <th>Average Math Score</th>
      <th>Average Reading Score</th>
      <th>School Type</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Ford High School</th>
      <td>2739</td>
      <td>1763916</td>
      <td>644.0</td>
      <td>68.309602</td>
      <td>79.299014</td>
      <td>73.804308</td>
      <td>77.102592</td>
      <td>80.746258</td>
      <td>District</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>4761</td>
      <td>3094650</td>
      <td>650.0</td>
      <td>66.057551</td>
      <td>81.222432</td>
      <td>73.639992</td>
      <td>77.072464</td>
      <td>80.966394</td>
      <td>District</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>2917</td>
      <td>1910635</td>
      <td>655.0</td>
      <td>65.683922</td>
      <td>81.316421</td>
      <td>73.500171</td>
      <td>76.629414</td>
      <td>81.182722</td>
      <td>District</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>2949</td>
      <td>1884411</td>
      <td>639.0</td>
      <td>65.988471</td>
      <td>80.739234</td>
      <td>73.363852</td>
      <td>76.711767</td>
      <td>81.158020</td>
      <td>District</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>3999</td>
      <td>2547363</td>
      <td>637.0</td>
      <td>66.366592</td>
      <td>80.220055</td>
      <td>73.293323</td>
      <td>76.842711</td>
      <td>80.744686</td>
      <td>District</td>
    </tr>
  </tbody>
</table>
</div>




```python
score_grade = students_df[["school", "grade", "math_score"]]
```


```python
grade_9th = score_grade.loc[score_grade["grade"] == "9th"]
grade_9th_group = grade_9th.groupby(["school"])
grade_9th_group_mean = grade_9th_group.mean()

grade_10th = score_grade.loc[score_grade["grade"] == "10th"]
grade_10th_group = grade_10th.groupby(["school"])
grade_10th_group_mean = grade_10th_group.mean()

grade_11th = score_grade.loc[score_grade["grade"] == "11th"]
grade_11th_group = grade_11th.groupby(["school"])
grade_11th_group_mean = grade_11th_group.mean()

grade_12th = score_grade.loc[score_grade["grade"] == "12th"]
grade_12th_group = grade_12th.groupby(["school"])
grade_12th_group_mean = grade_12th_group.mean()
```


```python
combined_grade_data_1 = pd.merge(grade_9th_group_mean, grade_10th_group_mean, left_index=True, right_index=True)
combined_grade_data_1 = combined_grade_data_1.rename(columns={"math_score_x":"9th", "math_score_y": "10th"})

combined_grade_data_2 = pd.merge(combined_grade_data_1, grade_11th_group_mean, left_index=True, right_index=True)
combined_grade_data_2 = combined_grade_data_2.rename(columns={"math_score":"11th"})

combined_grade_data_full = pd.merge(combined_grade_data_2, grade_12th_group_mean, left_index=True, right_index=True)
combined_grade_data_full = combined_grade_data_full.rename(columns={"math_score":"12th"})
combined_grade_data_full
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
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>77.083676</td>
      <td>76.996772</td>
      <td>77.515588</td>
      <td>76.492218</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.094697</td>
      <td>83.154506</td>
      <td>82.765560</td>
      <td>83.277487</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>76.403037</td>
      <td>76.539974</td>
      <td>76.884344</td>
      <td>77.151369</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>77.361345</td>
      <td>77.672316</td>
      <td>76.918058</td>
      <td>76.179963</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>82.044010</td>
      <td>84.229064</td>
      <td>83.842105</td>
      <td>83.356164</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>77.438495</td>
      <td>77.337408</td>
      <td>77.136029</td>
      <td>77.186567</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.787402</td>
      <td>83.429825</td>
      <td>85.000000</td>
      <td>82.855422</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>77.027251</td>
      <td>75.908735</td>
      <td>76.446602</td>
      <td>77.225641</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>77.187857</td>
      <td>76.691117</td>
      <td>77.491653</td>
      <td>76.863248</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.625455</td>
      <td>83.372000</td>
      <td>84.328125</td>
      <td>84.121547</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>76.859966</td>
      <td>76.612500</td>
      <td>76.395626</td>
      <td>77.690748</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>83.420755</td>
      <td>82.917411</td>
      <td>83.383495</td>
      <td>83.778976</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.590022</td>
      <td>83.087886</td>
      <td>83.498795</td>
      <td>83.497041</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.085578</td>
      <td>83.724422</td>
      <td>83.195326</td>
      <td>83.035794</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.264706</td>
      <td>84.010288</td>
      <td>83.836782</td>
      <td>83.644986</td>
    </tr>
  </tbody>
</table>
</div>




```python
reading_grade = students_df[["school", "grade", "reading_score"]]

reading_9th = reading_grade.loc[score_grade["grade"] == "9th"]
reading_9th_group = reading_9th.groupby(["school"])
reading_9th_group_mean = reading_9th_group.mean()

reading_10th = reading_grade.loc[score_grade["grade"] == "10th"]
reading_10th_group = reading_10th.groupby(["school"])
reading_10th_group_mean = reading_10th_group.mean()

reading_11th = reading_grade.loc[score_grade["grade"] == "11th"]
reading_11th_group = reading_11th.groupby(["school"])
reading_11th_group_mean = reading_11th_group.mean()

reading_12th = reading_grade.loc[score_grade["grade"] == "12th"]
reading_12th_group = reading_12th.groupby(["school"])
reading_12th_group_mean = reading_12th_group.mean()
```


```python
combined_reading_data_1 = pd.merge(reading_9th_group_mean, reading_10th_group_mean, left_index=True, right_index=True)
combined_reading_data_1 = combined_reading_data_1.rename(columns={"reading_score_x":"9th", "reading_score_y": "10th"})

combined_reading_data_2 = pd.merge(combined_reading_data_1, reading_11th_group_mean, left_index=True, right_index=True)
combined_reading_data_2 = combined_reading_data_2.rename(columns={"reading_score":"11th"})

combined_reading_data_full = pd.merge(combined_reading_data_2, reading_12th_group_mean, left_index=True, right_index=True)
combined_reading_data_full = combined_reading_data_full.rename(columns={"reading_score":"12th"})
combined_reading_data_full
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
      <th>9th</th>
      <th>10th</th>
      <th>11th</th>
      <th>12th</th>
    </tr>
    <tr>
      <th>school</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Bailey High School</th>
      <td>81.303155</td>
      <td>80.907183</td>
      <td>80.945643</td>
      <td>80.912451</td>
    </tr>
    <tr>
      <th>Cabrera High School</th>
      <td>83.676136</td>
      <td>84.253219</td>
      <td>83.788382</td>
      <td>84.287958</td>
    </tr>
    <tr>
      <th>Figueroa High School</th>
      <td>81.198598</td>
      <td>81.408912</td>
      <td>80.640339</td>
      <td>81.384863</td>
    </tr>
    <tr>
      <th>Ford High School</th>
      <td>80.632653</td>
      <td>81.262712</td>
      <td>80.403642</td>
      <td>80.662338</td>
    </tr>
    <tr>
      <th>Griffin High School</th>
      <td>83.369193</td>
      <td>83.706897</td>
      <td>84.288089</td>
      <td>84.013699</td>
    </tr>
    <tr>
      <th>Hernandez High School</th>
      <td>80.866860</td>
      <td>80.660147</td>
      <td>81.396140</td>
      <td>80.857143</td>
    </tr>
    <tr>
      <th>Holden High School</th>
      <td>83.677165</td>
      <td>83.324561</td>
      <td>83.815534</td>
      <td>84.698795</td>
    </tr>
    <tr>
      <th>Huang High School</th>
      <td>81.290284</td>
      <td>81.512386</td>
      <td>81.417476</td>
      <td>80.305983</td>
    </tr>
    <tr>
      <th>Johnson High School</th>
      <td>81.260714</td>
      <td>80.773431</td>
      <td>80.616027</td>
      <td>81.227564</td>
    </tr>
    <tr>
      <th>Pena High School</th>
      <td>83.807273</td>
      <td>83.612000</td>
      <td>84.335938</td>
      <td>84.591160</td>
    </tr>
    <tr>
      <th>Rodriguez High School</th>
      <td>80.993127</td>
      <td>80.629808</td>
      <td>80.864811</td>
      <td>80.376426</td>
    </tr>
    <tr>
      <th>Shelton High School</th>
      <td>84.122642</td>
      <td>83.441964</td>
      <td>84.373786</td>
      <td>82.781671</td>
    </tr>
    <tr>
      <th>Thomas High School</th>
      <td>83.728850</td>
      <td>84.254157</td>
      <td>83.585542</td>
      <td>83.831361</td>
    </tr>
    <tr>
      <th>Wilson High School</th>
      <td>83.939778</td>
      <td>84.021452</td>
      <td>83.764608</td>
      <td>84.317673</td>
    </tr>
    <tr>
      <th>Wright High School</th>
      <td>83.833333</td>
      <td>83.812757</td>
      <td>84.156322</td>
      <td>84.073171</td>
    </tr>
  </tbody>
</table>
</div>




```python
per_student_analysis = total_comparison_4[["Per Student Budget", "Average Math Score", "Average Reading Score", "% Passing Math", "% Passing Reading", "Overall Passing Rate"]]
per_student_analysis = per_student_analysis.reset_index(level=0, drop=True)

bins = [0, 599, 625, 649, 700]
group_names = ['< 600', '600-624', '625-649', '650 and up']

budget_series = pd.cut(per_student_analysis["Per Student Budget"], bins, labels=group_names)

per_student_analysis["Per Student Budget"] = budget_series

per_student_group = per_student_analysis.groupby("Per Student Budget")
per_student_group_mean = per_student_group.mean()
per_student_group_mean
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
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>Per Student Budget</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>&lt; 600</th>
      <td>83.455399</td>
      <td>83.933814</td>
      <td>93.460096</td>
      <td>96.610877</td>
      <td>95.035486</td>
    </tr>
    <tr>
      <th>600-624</th>
      <td>83.516957</td>
      <td>83.862393</td>
      <td>93.951362</td>
      <td>96.313180</td>
      <td>95.132271</td>
    </tr>
    <tr>
      <th>625-649</th>
      <td>78.224770</td>
      <td>81.506371</td>
      <td>72.123380</td>
      <td>83.900090</td>
      <td>78.011735</td>
    </tr>
    <tr>
      <th>650 and up</th>
      <td>76.997210</td>
      <td>81.027843</td>
      <td>66.164813</td>
      <td>81.133951</td>
      <td>73.649382</td>
    </tr>
  </tbody>
</table>
</div>




```python
school_size_analysis = total_comparison_4[["Total Students", "Average Math Score", "Average Reading Score", "% Passing Math", "% Passing Reading", "Overall Passing Rate"]]
school_size_analysis = school_size_analysis.reset_index(level=0, drop=True)

bins_2 = [0, 1500, 3000, 10000]
group_names_2 = ['Small', 'Medium', 'Large']

size_series = pd.cut(school_size_analysis["Total Students"], bins_2, labels=group_names_2)

school_size_analysis["Total Students"] = size_series

school_size_group = school_size_analysis.groupby("Total Students")
school_size_group_mean = school_size_group.mean()
school_size_group_mean
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
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
    </tr>
    <tr>
      <th>Total Students</th>
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
      <td>83.664898</td>
      <td>83.892148</td>
      <td>93.497607</td>
      <td>96.445946</td>
      <td>94.971776</td>
    </tr>
    <tr>
      <th>Medium</th>
      <td>80.904987</td>
      <td>82.822740</td>
      <td>83.556977</td>
      <td>90.588593</td>
      <td>87.072785</td>
    </tr>
    <tr>
      <th>Large</th>
      <td>77.063340</td>
      <td>80.919864</td>
      <td>66.464293</td>
      <td>81.059691</td>
      <td>73.761992</td>
    </tr>
  </tbody>
</table>
</div>




```python
school_type_analysis = total_comparison_4[["School Type", "Average Math Score", "Average Reading Score", "% Passing Math", "% Passing Reading", "Overall Passing Rate"]]
school_type_analysis = school_type_analysis.reset_index(level=0, drop=True)

school_type_group = school_type_analysis.groupby("School Type")
school_type_group_mean = school_type_group.mean()
school_type_group_mean
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
      <th>% Passing Math</th>
      <th>% Passing Reading</th>
      <th>Overall Passing Rate</th>
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
      <td>83.473852</td>
      <td>83.896421</td>
      <td>93.620830</td>
      <td>96.586489</td>
      <td>95.103660</td>
    </tr>
    <tr>
      <th>District</th>
      <td>76.956733</td>
      <td>80.966636</td>
      <td>66.548453</td>
      <td>80.799062</td>
      <td>73.673757</td>
    </tr>
  </tbody>
</table>
</div>


