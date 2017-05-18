# Data Challenge Solution Doc

This document provides with you step by step instruction about how to import and use Load_Insti_Info.py which performs data exporting, quality checks and analytical visulization; 

Anaconda 4.3.1 python is used for scripting;
packages required are as follow:

```
import os
import seaborn as sns 
import pandas as pd
from sqlalchemy import create_engine
import datetime as dt
```
## Getting Started 
### QUATION 1: hmda_init() & hmda_to_json(data, states, conventional_conforming)

#### Step1:
unzip and place Load_Insti_Info.py the same place with two data files: 2012_to_2014_institutions_data.csv and 2012_to_2014_loans_data.csv

#### Step2:
open a python IDLE and import Load_Insti_Info

```
import os
curr_dir='/where/is/the/csv/data/located'  # customize the dir value accordingly
os.chdir(curr_dir)
from Load_Insti_Info import *
```
Note: 'from Load_Insti_Info import *' will internally read the two csv files and load into two sqlite tables and compile two functions:
hmda_init() & hmda_to_json(data, states, conventional_conforming) two be used for further steps;

#### Step3:
Merge two dataset according and transform the result as a dataframe format
```
data=hmda_init()
```
#### Step4:
To use hmda_to_json() function to export the data accordingly with optional parameter as filters:
```
log_output=hmda_to_json(data,output_nm='data.json',states='VA',conventional_conforming='all',year='2012,2013,2018')
print log_output

'''
Function hmda_to_json() explanation:
this function print out dataframe input to json formatted output based on parameter values you configured.
log_output is a return var of the function which stores log info.

Parameter list:
# data: input dataframe
# output_nm: give a name for output json file; output dir will be working directory
# states: optional parameter with state name, case insensitive and delimited by ',', eg: 'VA,MD'; input 'ALL' if want all states
# conventional_conforming: optional parameter with valid inputs: 'ALL', 'Y', 'N'. case insenstive
# year: optional parameter, case insensitive and delimited by ',', eg: '2012,2014'; input 'ALL' if want all years
'''
```

### QUATION 2: data quality checks

```
# DATA ISSUE1: return not-unique respondent name per respondent ID
t1=data[['RESPONDENT_ID','RESPONDENT_NAME_TS']].drop_duplicates()
t2 = t1.groupby('RESPONDENT_ID')['RESPONDENT_NAME_TS'].size()
t3=t2[t2 > 1]
dup_id=[str(m) for m in t3.index.values.tolist()]
dup_id_nm=data[data['RESPONDENT_ID'].isin(dup_id)][['RESPONDENT_ID','RESPONDENT_NAME_TS']].drop_duplicates().sort_values(['RESPONDENT_ID','RESPONDENT_NAME_TS'])

# DATA ISSUE2: some numeric cols (eg: TRACT_TO_MSA_MD_INCOME_PCT,FFIEC_MEDIAN_FAMILY_INCOME) have text value 'NA'
# Example 1
d1=data['TRACT_TO_MSA_MD_INCOME_PCT'].drop_duplicates().values.tolist()
d2=[str(m) for m in d1]
invalid_v1=[m for m in d2 if not str.isdigit(m.strip('0').replace('.',''))]
print invalid_v1

# Example 2
d1=data['FFIEC_MEDIAN_FAMILY_INCOME'].drop_duplicates().values.tolist()
d2=[str(m) for m in d1]
invalid_v1=[m for m in d2 if not str.isdigit(m.strip('0').replace('.',''))]
print invalid_v1


# Loan_Amount_000 - did not find any data quality issue; all rounded to integer
d1=data['LOAN_AMOUNT_000'].drop_duplicates().values.tolist()
d2=[str(m) for m in d1]
invalid_v1=[m for m in d2 if not str.isdigit(m.strip('0').replace('.',''))]
print invalid_v1
```

### QUATION 3: visulization

For a detailed answer for this question, please review file: question3_business_strategy_analysis.docx
code for graphs used in the doc is as follow:

```
data1=data

# Graph 1
g_sum=data1.groupby(['AS_OF_YEAR', 'STATE'])['LOAN_AMOUNT_000'].sum()
g_sum_f=g_sum.to_frame()
g_sum_f.reset_index(level=0, inplace=True)
g_sum_f['STATE']=g_sum_f.index
g_sum_tot=g_sum_f
g = sns.factorplot(x="AS_OF_YEAR", y="LOAN_AMOUNT_000", col="STATE",\
                 data=g_sum_f, saturation=.5,\
                 kind="bar", ci=None, aspect=.6)
(g.set_axis_labels("", "sum of loan")\
.set_xticklabels(["2012", "2013", "2014"])\
.set_titles("{col_name} {col_var}")\
.set(ylim=(0, 270000000))\
.despine(left=True))
sns.plt.show()


# Graph 2
data1['APPLI_ID']=data1['RESPONDENT_ID']+data1['SEQUENCE_NUMBER'].map(str)
g_sum=data1.groupby(['AS_OF_YEAR', 'STATE'])['APPLI_ID'].nunique()
g_sum_f=g_sum.to_frame()
g_sum_f.reset_index(level=0, inplace=True)
g_sum_f['STATE']=g_sum_f.index
g_cnt_appli=g_sum_f
g = sns.factorplot(x="AS_OF_YEAR", y="APPLI_ID", col="STATE",\
                    data=g_sum_f, saturation=.5,\
                    kind="bar", ci=None, aspect=.6)

(g.set_axis_labels("", "Avg of loan")\
.set_xticklabels(["2012", "2013", "2014"])\
.set_titles("{col_name} {col_var}")\
.set(ylim=(0, 250000))\
.despine(left=True))
sns.plt.show()


# Graph 3
#data['APPLI_ID']=data['RESPONDENT_ID']+data['SEQUENCE_NUMBER'].map(str)
g_sum=data1.groupby(['AS_OF_YEAR', 'STATE'])['RESPONDENT_ID'].nunique()
g_sum_f=g_sum.to_frame()
g_sum_f.reset_index(level=0, inplace=True)
g_sum_f['STATE']=g_sum_f.index
g_dist_insti=g_sum_f
g = sns.factorplot(x="AS_OF_YEAR", y="RESPONDENT_ID", col="STATE",\
                    data=g_sum_f, saturation=.5,\
                    kind="bar", ci=None, aspect=.6)
(g.set_axis_labels("", "Avg of loan")\
.set_xticklabels(["2012", "2013", "2014"])\
.set_titles("{col_name} {col_var}")\
.set(ylim=(0, 900))\
.despine(left=True))
sns.plt.show()

# Graph 4
#data['APPLI_ID']=data['RESPONDENT_ID']+data['SEQUENCE_NUMBER'].map(str)
result = pd.merge(g_dist_insti, g_sum_tot, how='left',on=['STATE', 'AS_OF_YEAR'])
result['RATIO']=result['LOAN_AMOUNT_000'].map(int)/result['RESPONDENT_ID'].map(int)
g = sns.factorplot(x="AS_OF_YEAR", y="RATIO", col="STATE",\
                    data=result, saturation=.5,\
                    kind="bar", ci=None, aspect=.6)

(g.set_axis_labels("", "Appli cnt per Insti")\
.set_xticklabels(["2012", "2013", "2014"])\
.set_titles("{col_name} {col_var}")\
.set(ylim=(0, 300000))\
.despine(left=True))
sns.plt.show()
```

## contact
haoding88@gmail.com
(213)-290-7047
Hao Ding


Many many thanks for review;
