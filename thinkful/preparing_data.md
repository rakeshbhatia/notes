
# **Preparing Data**


```python
import pandas as pd
from matplotlib import pyplot as plt
import numpy as np
from sklearn import linear_model
%matplotlib inline
```


```python
ny_crime = pd.read_csv('table_8_offenses_known_to_law_enforcement_new_york_by_city_2013.csv')

ny_crime.head()
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
      <th>City</th>
      <th>Population</th>
      <th>Violentcrime</th>
      <th>Murder andnonnegligentmanslaughter</th>
      <th>Rape(reviseddefinition)1</th>
      <th>Rape(legacydefinition)2</th>
      <th>Robbery</th>
      <th>Aggravatedassault</th>
      <th>Propertycrime</th>
      <th>Burglary</th>
      <th>Larceny-theft</th>
      <th>Motorvehicletheft</th>
      <th>Arson3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Adams Village</td>
      <td>1,861</td>
      <td>0</td>
      <td>0</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>12</td>
      <td>2</td>
      <td>10</td>
      <td>0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Addison Town and Village</td>
      <td>2,577</td>
      <td>3</td>
      <td>0</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>24</td>
      <td>3</td>
      <td>20</td>
      <td>1</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Akron Village</td>
      <td>2,846</td>
      <td>3</td>
      <td>0</td>
      <td>NaN</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>16</td>
      <td>1</td>
      <td>15</td>
      <td>0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Albany</td>
      <td>97,956</td>
      <td>791</td>
      <td>8</td>
      <td>NaN</td>
      <td>30</td>
      <td>227</td>
      <td>526</td>
      <td>4,090</td>
      <td>705</td>
      <td>3,243</td>
      <td>142</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Albion Village</td>
      <td>6,388</td>
      <td>23</td>
      <td>0</td>
      <td>NaN</td>
      <td>3</td>
      <td>4</td>
      <td>16</td>
      <td>223</td>
      <td>53</td>
      <td>165</td>
      <td>5</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Select datafame columns
ny_crime = ny_crime[['City', 'Population', 'Murder and\rnonnegligent\rmanslaughter', 'Robbery', 'Property\rcrime']]

# Rename columns
ny_crime.columns = ['City', 'Population', 'Murder', 'Robbery', 'Property Crime']

# Remove commas from numeric strings
ny_crime['Population'] = ny_crime.Population.apply(lambda x: x.replace(',', ''))

# Change type to int
ny_crime['Population'] = ny_crime.Population.astype(int)

# Remove commas from numeric strings
ny_crime['Robbery'] = ny_crime.Robbery.apply(lambda x: x.replace(',', ''))

# Change type to int
ny_crime['Robbery'] = ny_crime.Robbery.astype(int)

# Remove commas from numeric strings
ny_crime['Property Crime'] = ny_crime['Property Crime'].apply(lambda x: x.replace(',', ''))

# Change type to int
ny_crime['Property Crime'] = ny_crime['Property Crime'].astype(int)

# Drop null values
ny_crime = ny_crime.dropna()

ny_crime.head()
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
      <th>City</th>
      <th>Population</th>
      <th>Murder</th>
      <th>Robbery</th>
      <th>Property Crime</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Adams Village</td>
      <td>1861</td>
      <td>0</td>
      <td>0</td>
      <td>12</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Addison Town and Village</td>
      <td>2577</td>
      <td>0</td>
      <td>0</td>
      <td>24</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Akron Village</td>
      <td>2846</td>
      <td>0</td>
      <td>0</td>
      <td>16</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Albany</td>
      <td>97956</td>
      <td>8</td>
      <td>227</td>
      <td>4090</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Albion Village</td>
      <td>6388</td>
      <td>0</td>
      <td>4</td>
      <td>223</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Plot population distribution
ny_crime['Population'].hist(bins=100)
plt.title('Population')
plt.show()
```


![png](Preparing%20Data_files/Preparing%20Data_4_0.png)



```python
# Plot murder distribution
ny_crime['Murder'].hist(bins=50)
plt.title('Murder')
plt.show()
```


![png](Preparing%20Data_files/Preparing%20Data_5_0.png)



```python
# Plot robbery distribution
ny_crime['Robbery'].hist(bins=50)
plt.title('Robbery')
plt.show()
```


![png](Preparing%20Data_files/Preparing%20Data_6_0.png)



```python
# Plot property crime distribution
ny_crime['Property Crime'].hist(bins=50)
plt.title('Murder')
plt.show()
```


![png](Preparing%20Data_files/Preparing%20Data_7_0.png)



```python
# Filter out any outliers over two standard deviations above the mean
pop_cutoff = ny_crime['Population'].mean() + 2*ny_crime['Population'].std()
mur_cutoff = ny_crime['Murder'].mean() + 2*ny_crime['Murder'].std()
rob_cutoff = ny_crime['Robbery'].mean() + 2*ny_crime['Robbery'].std()
prop_cutoff = ny_crime['Property Crime'].mean() + 2*ny_crime['Property Crime'].std()

ny_crime['Population'] = ny_crime.Population.map(lambda x: x if x < pop_cutoff else None)
ny_crime['Murder'] = ny_crime.Murder.map(lambda x: x if x < mur_cutoff else None)
ny_crime['Robbery'] = ny_crime.Robbery.map(lambda x: x if x < rob_cutoff else None)
ny_crime['Property Crime'] = ny_crime['Property Crime'].map(lambda x: x if x < prop_cutoff else None)

ny_crime.describe()
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
      <th>Population</th>
      <th>Murder</th>
      <th>Robbery</th>
      <th>Property Crime</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>347.000000</td>
      <td>345.000000</td>
      <td>347.000000</td>
      <td>347.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>15956.685879</td>
      <td>0.350725</td>
      <td>17.867435</td>
      <td>385.752161</td>
    </tr>
    <tr>
      <th>std</th>
      <td>27080.218837</td>
      <td>1.587160</td>
      <td>94.972492</td>
      <td>1034.369072</td>
    </tr>
    <tr>
      <th>min</th>
      <td>526.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>2997.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>40.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>7187.000000</td>
      <td>0.000000</td>
      <td>1.000000</td>
      <td>112.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>18160.500000</td>
      <td>0.000000</td>
      <td>5.000000</td>
      <td>340.500000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>258789.000000</td>
      <td>21.000000</td>
      <td>1322.000000</td>
      <td>12491.000000</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Create new feature
ny_crime['Population^2'] = ny_crime['Population']**2

# Convert specified columns to boolean
for col in ['Murder', 'Robbery']:
    ny_crime[col] = ny_crime[col] > 0

ny_crime.head()
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
      <th>City</th>
      <th>Population</th>
      <th>Murder</th>
      <th>Robbery</th>
      <th>Property Crime</th>
      <th>Population^2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Adams Village</td>
      <td>1861.0</td>
      <td>False</td>
      <td>False</td>
      <td>12.0</td>
      <td>3.463321e+06</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Addison Town and Village</td>
      <td>2577.0</td>
      <td>False</td>
      <td>False</td>
      <td>24.0</td>
      <td>6.640929e+06</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Akron Village</td>
      <td>2846.0</td>
      <td>False</td>
      <td>False</td>
      <td>16.0</td>
      <td>8.099716e+06</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Albany</td>
      <td>97956.0</td>
      <td>True</td>
      <td>True</td>
      <td>4090.0</td>
      <td>9.595378e+09</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Albion Village</td>
      <td>6388.0</td>
      <td>False</td>
      <td>True</td>
      <td>223.0</td>
      <td>4.080654e+07</td>
    </tr>
  </tbody>
</table>
</div>




```python

```
