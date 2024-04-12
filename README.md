

<img src="[Assets/icon.png](https://github.com/srisainee/HCAHPS-Patient-Survey/blob/main/U.S.%20HCAHPS%20Survey%20Analysis.png)" width="200">

# U.S HCAHPS Survey Analysis

The American Hospital Association (AHA), a national organization that represents hospitals and their patients, and acts as a source of information on health care issues and trends. The dataset is the Hospital Consumer Assessment of Healthcare Providers and Systems (HCAHPS) survey results for the last 9 years provided by [Maven Analytics](https://mavenanalytics.io/challenges/maven-healthcare-challenge/26).  

The surveys contains questions to evaluate the following measures:

    Communication with Nurses - H_COMP_1
    Communication with Doctors - H_COMP_2
    Responsiveness of Hospital Staff - H_COMP_3
    Communication about Medicines - H_COMP_5
    Discharge Information - H_COMP_6
    Care Transition - H_COMP_7
    Cleanliness of Hospital Environment - H_CLEAN_HSP
    Quietness of Hospital Environment - H_QUITE_HSP
    Overall Hospital Rating - H_HSP_RATING
    Willingness to Recommend the Hospital - H_RECMND
    

    
### Purpose of analysis
The aim of this analysis is to examine the factors influencing the rankings of hospitals across the states, first, by  testing the hypothesis if the states with higher number of facilities have higher ranking of the hospitals.

### Approach
The survey results in the format 
    
    Bottom-box Answer	Middle-box Answer	Top-box Answer
    Sometimes or never   Usually	          Always

Each record of the national survey results is represented in percentage and sums up to 100%. Therefore, "Top-box Answer" of the result is chosen to test the hypothesis. The dataset is cleaned to merge with the expenditure data. 


```python
# importing libraries and data
import pandas as pd
import numpy as np 
from matplotlib import pyplot
```

To investigate the relationship between the number of facilities in a state and the average ranking of hospitals in that state, Spearman's correlation test can be used as we are trying to find the relationship between 2 continuous variables. The assumption in this case is that the relationship is non linear and not normally distributed. Therefore, Spearman's rank correlation is a suitable choice as it is a non-parametric test that assesses the strength and direction of a monotonic relationship.

###### Hypotheses:

    Null Hypothesis: There is no correlation between the number of facilities and the average ranking of hospitals in
    the states.
    Alternate Hypothesis: There is a correlation between the number of facilities and the average ranking of hospitals in 
    the states.


```python
from scipy.stats import spearmanr

states_results = pd.read_csv('./HCAHPS+Patient+Survey/data_tables/state_results.csv')
responses = pd.read_csv('./HCAHPS+Patient+Survey/data_tables/responses.csv')

# grouping to get number of facilities surveyed by state
facilities = responses.groupby(['Release Period', 'State']).agg({'Facility ID': 'nunique'}).reset_index()

# exracting average hospital ratings of each state. 
states = states_results[states_results['Measure ID'] == 'H_HSP_RATING']
states = states[['Release Period','State', 'Top-box Percentage']]

df = pd.merge(states, facilities, how='inner',  
              left_on=['Release Period', 'State'], right_on=['Release Period', 'State'])

x1 = df[['Facility ID']]
x2 = df[['Top-box Percentage']]

corr, p = spearmanr(x1, x2)
print('Spearman correlation coefficient is ' + str(round(corr,3)) )
print('Spearman p value is ' + str(round(p,3)) )

if(p<0.05):
    print("Significant relationship")
```

    Spearman correlation coefficient is 0.217
    Spearman p value is 0.0
    Significant relationship
    

A statistically significant correlation is observed which indicates that we reject the null hypothesis as there is sufficient evidence to claim that there is a significant relationship between the number of facilities in ths state and the ranking of the hospitals in the state. 

A positive correlation coefficient indicates a positive relationship as when the number of facilities in a state increases, the ranking of the state also increases. However, the being coefficient close to zero suggests a weak or no linear relationship. 

In this case, it is essential to consider both pieces of information together. A weak correlation may not be practically significant if it is statistically significant. Additionally, correlation does not imply causation, and other factors should be considered in a comprehensive analysis of the relationship between variables.

To further investigate the factors that affect the rankings of the hospitals, the relationship between healthcare spending and the ranking of hospitals will be investigated. 

## Does health care spending correlate to patient satisfaction and ranking of hospitals across different states?

To test the hypothesis if the spending on healthcare influence the ranking of the hospitals Per Person State Public Health Funding data is sourced from the [State Health Access Data Assistance Center](https://statehealthcompare.shadac.org/Bulk#1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,43,44,45,46,47,48,49,50,51,52/117). 
This dataset was selected due to its documentation of per capita public healthcare funding by state and alignment with the survey period, both tracking the fiscal year which is expected to yield more precise outcomes [[1]](https://www.investopedia.com/terms/f/fiscalyear.asp#:~:text=For%20example%2C%20the%20U.S.%20government's,channels%20to%20Congress%20for%20approval).
 
Since the hypothesis is investigating relationship between 2 continous variables - the state wise health expenditure and the hospital ranking in the states, Spearman’s Correlation test will be used again.  

In addition to the overall hospital ranking, the realtionship between the health expenditure and each of the measure surveyed is tested to prove if there is a statistically significant relationship between the ranking and the evaluation measures. 

###### Hypothesis 
    Null Hypothesis: There is no correlation between the 2 variables
    Alternate Hypothesis: There is a linear relationship between the 2 variables i.e, the measure increases and/or decreases with the health spend.


```python
state_spends = pd.read_csv('./Shadac-BulkData-03122024/Per capita state public health funding.csv')
state_spends = state_spends.rename(columns={'Location':'State', 'TimeFrame': 'Year', 'Data': 'Funding'})
state_spends = state_spends[['Year', 'State', 'Funding']]
state_spends = state_spends.fillna(method='ffill')

state_abbr = pd.read_csv('./HCAHPS+Patient+Survey/data_tables/states.csv')
states_results = pd.merge(states_results, state_abbr, how='inner', left_on='State', right_on='State')
states_results['Year'] = states_results['Release Period'].str.strip().str[-4:]
states_results['Year'] = states_results['Year'].astype(int)
states_results['Year'] = states_results['Year'] -1
states_results = states_results[['Year', 'State Name', 'Measure ID','Top-box Percentage']]
states_results = states_results.rename(columns={'State Name':'State'})

state_data = pd.merge(states_results, state_spends, how='inner', left_on=['Year', 'State'], right_on=['Year', 'State'])
state_data
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Year</th>
      <th>State</th>
      <th>Measure ID</th>
      <th>Top-box Percentage</th>
      <th>Funding</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2014</td>
      <td>Alaska</td>
      <td>H_CLEAN_HSP</td>
      <td>70</td>
      <td>105.58</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2014</td>
      <td>Alaska</td>
      <td>H_COMP_1</td>
      <td>74</td>
      <td>105.58</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2014</td>
      <td>Alaska</td>
      <td>H_COMP_2</td>
      <td>75</td>
      <td>105.58</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2014</td>
      <td>Alaska</td>
      <td>H_COMP_3</td>
      <td>68</td>
      <td>105.58</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2014</td>
      <td>Alaska</td>
      <td>H_COMP_5</td>
      <td>64</td>
      <td>105.58</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>3985</th>
      <td>2021</td>
      <td>Wyoming</td>
      <td>H_COMP_6</td>
      <td>88</td>
      <td>27.23</td>
    </tr>
    <tr>
      <th>3986</th>
      <td>2021</td>
      <td>Wyoming</td>
      <td>H_COMP_7</td>
      <td>54</td>
      <td>27.23</td>
    </tr>
    <tr>
      <th>3987</th>
      <td>2021</td>
      <td>Wyoming</td>
      <td>H_HSP_RATING</td>
      <td>71</td>
      <td>27.23</td>
    </tr>
    <tr>
      <th>3988</th>
      <td>2021</td>
      <td>Wyoming</td>
      <td>H_QUIET_HSP</td>
      <td>66</td>
      <td>27.23</td>
    </tr>
    <tr>
      <th>3989</th>
      <td>2021</td>
      <td>Wyoming</td>
      <td>H_RECMND</td>
      <td>69</td>
      <td>27.23</td>
    </tr>
  </tbody>
</table>
<p>3990 rows × 5 columns</p>
</div>




```python
measures = ['H_COMP_1','H_COMP_2','H_COMP_3','H_COMP_5','H_COMP_6','H_COMP_7','H_CLEAN_HSP',
            'H_QUIET_HSP','H_HSP_RATING','H_RECMND']

for measure in measures :
    df = state_data[state_data['Measure ID'] == measure]
    
    x1 = df[['Funding']]
    x2 = df[['Top-box Percentage']]

    corr, p = spearmanr(x1, x2)

    if(p<0.05):
        print('Spearman correlation for measure ' + measure + ' is ' + str(round(corr,3)) )
        print('Spearman p value for measure ' + measure + ' is ' + str(round(p,3)) )
```

    Spearman correlation for measure H_QUIET_HSP is -0.102
    Spearman p value for measure H_QUIET_HSP is 0.042
    Spearman correlation for measure H_HSP_RATING is -0.106
    Spearman p value for measure H_HSP_RATING is 0.034
    

The results of the Spearman's correlation tests suggests that there is statistical significance correlation between the per capital health spend of the states and the ranking and quietness of the hospital facilities in that state. However, the coefficient of test indicates a weak negative correlation between per capita healthcare spending and the ranking and quietness of facilities in the state. This indicates an inverse relationship as per capita healthcare spending increases, the ranking and quietness of facilities tends to decrease, and vice versa. 

This result suggests an association between the variables but does not necessarily indicate a causal relationship.

To further investigate the factors influencing the ranking, the relationship between the evaluation measures themselves and the ranking of the hospitals in that state is tested. 

## Do higher scores in the evaluation measures correlate with increased ranking scores?


```python
measures = ['H_COMP_1', 'H_COMP_2', 'H_COMP_3', 'H_COMP_5', 'H_COMP_6', 'H_COMP_7', 'H_CLEAN_HSP', 'H_QUIET_HSP', 'H_RECMND']
target_measure = 'H_HSP_RATING'

df = state_data.pivot_table(index=['Year', 'State'], columns='Measure ID', values='Top-box Percentage').reset_index()

target_measure_values = df[target_measure]
measures_values_to_compare = df[measures]

# Calculate Spearman's correlation coefficients and p-values between the target measure and measures to compare
# correlation_results = {}
for measure in measures:
    corr, p = spearmanr(measures_values_to_compare[measure], target_measure_values)
    if(p<0.05):
        print('Spearman correlation for measure ' + measure + ' is ' + str(round(corr,3)) )
        print('Spearman p value for measure ' + measure + ' is ' + str(round(p,3)) )
```

    Spearman correlation for measure H_COMP_1 is 0.84
    Spearman p value for measure H_COMP_1 is 0.0
    Spearman correlation for measure H_COMP_2 is 0.748
    Spearman p value for measure H_COMP_2 is 0.0
    Spearman correlation for measure H_COMP_3 is 0.775
    Spearman p value for measure H_COMP_3 is 0.0
    Spearman correlation for measure H_COMP_5 is 0.747
    Spearman p value for measure H_COMP_5 is 0.0
    Spearman correlation for measure H_COMP_6 is 0.62
    Spearman p value for measure H_COMP_6 is 0.0
    Spearman correlation for measure H_COMP_7 is 0.886
    Spearman p value for measure H_COMP_7 is 0.0
    Spearman correlation for measure H_CLEAN_HSP is 0.836
    Spearman p value for measure H_CLEAN_HSP is 0.0
    Spearman correlation for measure H_QUIET_HSP is 0.601
    Spearman p value for measure H_QUIET_HSP is 0.0
    Spearman correlation for measure H_RECMND is 0.877
    Spearman p value for measure H_RECMND is 0.0
    

From the results it can be observed that a statistically significant association exists between all measures and the hospital facility ratings across states. This relationship is positivity strong, implying that as the scores for each measure increase, the ranking of facilities within states also increase.

In conclusion, neither the number of facilities nor healthcare expenditure directly impacts hospital ratings. However, the analysis suggests that each of the other measures influence these ratings.
