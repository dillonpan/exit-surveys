# exit-surveys
Project using the Python package "pandas" and various functions to combine, clean, and analyze data in multiple CSV files  

# Project Details:
For this project, we will take a look at employee exit surveys from two departments of the Queensland government in Australia:  
- Department of Education, Training and Employment (DETE)  
- Technical and Further Education (TAFE)  

Our main objective is to answer the following questions for both departments:
- Did employees who worked a for short period of time resign due to some kind of dissatisfaction?  
- What about employees who have been there longer?

# Note: Please replace [directory] below in the open() function with the link to your folder of choice where autos.csv is located

# Opening and Exploring the Data:
```python
import pandas
import numpy

dete_survey = pandas.read_csv('[directory].dete_survey.csv')
tafe_survey = pandas.read_csv("tafe_survey.csv")
print(len(dete_survey.columns))
print(len(tafe_survey.columns))
```
56  
72  

We can see that both CSV files contain a lot of columns, more than we probably need. Looking over both datasets, there's a couple of noticeable things we have to consider:  
1. The dete_survey dataset contains 'Not Stated' string values instead of NaN values  
2. Each dataframe contains many of the same columns, but the column names are different  
3. The data has multiple columns/rows areas that signify employee dissatisfaction lead to their resignation, so the datasets are a bit messy

# Identify Missing Values and Drop Unneccessary Columns
There's many steps we need to take to clean the data but we can start with the 'Not Stated' values and getting rid of uneccesary columns.
```python
# Read the file again but overwrite the 'Not Stated' cells with NaN values. Override the dete_survey variable.
dete_survey = pandas.read_csv('dete_survey.csv', na_values='Not Stated')
```

Let's assume that after looking over the datasets, we know which rows can be removed:
```python
# axis=1 is an optional argument stating to remove by columns. Default is axis=0, aka remove by row
dete_survey_updated = dete_survey.drop(dete_survey.columns[28:49], axis=1)
tafe_survey_updated = tafe_survey.drop(tafe_survey.columns[17:66], axis=1)

print(dete_survey_updated.columns)
print(tafe_survey_updated.columns)
```
Index(['ID', 'SeparationType', 'Cease Date', 'DETE Start Date',  
       'Role Start Date', 'Position', 'Classification', 'Region',  
       'Business Unit', 'Employment Status', 'Career move to public sector',  
       'Career move to private sector', 'Interpersonal conflicts',  
       'Job dissatisfaction', 'Dissatisfaction with the department',  
       'Physical work environment', 'Lack of recognition',  
       'Lack of job security', 'Work location', 'Employment conditions',  
       'Maternity/family', 'Relocation', 'Study/Travel', 'Ill Health',  
       'Traumatic incident', 'Work life balance', 'Workload',  
       'None of the above', 'Gender', 'Age', 'Aboriginal', 'Torres Strait',  
       'South Sea', 'Disability', 'NESB'],  
      dtype='object')  
      
Index(['Record ID', 'Institute', 'WorkArea', 'CESSATION YEAR',  
       'Reason for ceasing employment',  
       'Contributing Factors. Career Move - Public Sector ',  
       'Contributing Factors. Career Move - Private Sector ',  
       'Contributing Factors. Career Move - Self-employment',  
       'Contributing Factors. Ill Health',  
       'Contributing Factors. Maternity/Family',  
       'Contributing Factors. Dissatisfaction',  
       'Contributing Factors. Job Dissatisfaction',  
       'Contributing Factors. Interpersonal Conflict',  
       'Contributing Factors. Study', 'Contributing Factors. Travel',  
       'Contributing Factors. Other', 'Contributing Factors. NONE',  
       'Gender. What is your Gender?', 'CurrentAge. Current Age',  
       'Employment Type. Employment Type', 'Classification. Classification',  
       'LengthofServiceOverall. Overall Length of Service at Institute (in years)',  
       'LengthofServiceCurrent. Length of Service at current workplace (in years)'],  
      dtype='object')  
      
 # Rename Columns
Upon looking over the column headers for both datasets, we need to change some column headers from camelcase to snakecase. Afterwards, we need to get headers for both datasets to match so that we can merge the data later.

First off, let's  revise the column headers in dete_survey_updated to snakecase:
```python
dete_survey_updated.columns = dete_survey_updated.columns.str.lower().str.strip().str.replace(' ', '_')
print(dete_survey_updated.columns)
```

Index(['id', 'separationtype', 'cease_date', 'dete_start_date',  
       'role_start_date', 'position', 'classification', 'region',  
       'business_unit', 'employment_status', 'career_move_to_public_sector',  
       'career_move_to_private_sector', 'interpersonal_conflicts',  
       'job_dissatisfaction', 'dissatisfaction_with_the_department',  
       'physical_work_environment', 'lack_of_recognition',  
       'lack_of_job_security', 'work_location', 'employment_conditions',  
       'maternity/family', 'relocation', 'study/travel', 'ill_health',  
       'traumatic_incident', 'work_life_balance', 'workload',  
       'none_of_the_above', 'gender', 'age', 'aboriginal', 'torres_strait',  
       'south_sea', 'disability', 'nesb'],  
      dtype='object')  
      
Now we can update the column headers in tafe_survey_updated to match:
```python
mapping = {'Record ID': 'id', 'CESSATION YEAR': 'cease_date', 'Reason for ceasing employment': 'separationtype', 'Gender. What is your Gender?': 'gender', 'CurrentAge. Current Age': 'age',
       'Employment Type. Employment Type': 'employment_status',
       'Classification. Classification': 'position',
       'LengthofServiceOverall. Overall Length of Service at Institute (in years)': 'institute_service',
       'LengthofServiceCurrent. Length of Service at current workplace (in years)': 'role_service'}
tafe_survey_updated = tafe_survey_updated.rename(mapping, axis = 1)

print(tafe_survey_updated.columns)
```

Index(['id', 'Institute', 'WorkArea', 'cease_date', 'separationtype',  
       'Contributing Factors. Career Move - Public Sector ',  
       'Contributing Factors. Career Move - Private Sector ',  
       'Contributing Factors. Career Move - Self-employment',  
       'Contributing Factors. Ill Health',  
       'Contributing Factors. Maternity/Family',  
       'Contributing Factors. Dissatisfaction',  
       'Contributing Factors. Job Dissatisfaction',  
       'Contributing Factors. Interpersonal Conflict',  
       'Contributing Factors. Study', 'Contributing Factors. Travel',  
       'Contributing Factors. Other', 'Contributing Factors. NONE', 'gender',  
       'age', 'employment_status', 'position', 'institute_service',  
       'role_service'],  
      dtype='object')  

# Filter and Clean the Data

# Part1: Seperating Resignations From the Rest
In the beginning of this project, we specified that we wanted to analyze dissatisfaction on employees who retired. Let's see if there's a way to filter by those who resigned.

```python
print(dete_survey_updated['separationtype'].value_counts())
```
Age Retirement                          285  
Resignation-Other reasons               150  
Resignation-Other employer               91  
Resignation-Move overseas/interstate     70  
Voluntary Early Retirement (VER)         67  
Ill Health Retirement                    61  
Other                                    49  
Contract Expired                         34  
Termination                              15  
Name: separationtype, dtype: int64  

While it's easy to tell which values represent resignation, we do see that there's three different types of resignation. We will have to find a way to make sure we get all three types when filtering.

```python
print(tafe_survey_updated['separationtype'].value_counts())
```
Resignation                 340  
Contract Expired            127  
Retrenchment/ Redundancy    104  
Retirement                   82  
Transfer                     25  
Termination                  23  
Name: separationtype, dtype: int64  

This dataset is much easier to filter as there just one type of value we need to worry about, "Resignation". Thus we do not need to do much for the tafe_survey_updated dataset, but lets fix the dete_survey_updated dataset to match it. One way is to filter out those cells with the word "Resignation" in them then replace it with the actual word, "Resignation".

```python
dete_survey_updated.loc[dete_survey_updated['separationtype'].str.contains('Resignation'), 'separationtype'] = 'Resignation'

print(dete_survey_updated['separationtype'].value_counts())
```
Resignation                         311  
Age Retirement                      285  
Voluntary Early Retirement (VER)     67  
Ill Health Retirement                61  
Other                                49  
Contract Expired                     34  
Termination                          15  
Name: separationtype, dtype: int64  

Perfect, all three types of resignation were replaced and consolidated under "Resignation". Now we can filter both datasets and remove all the non-resignation rows:

```python
dete_resignations = dete_survey_updated[dete_survey_updated['separationtype'] == 'Resignation'].copy()
tafe_resignations = tafe_survey_updated[tafe_survey_updated['separationtype'] == 'Resignation'].copy()
```

# Part2: Cleaning Dates and Creating a New Column

Now that we a two datasets of only resignations, we need to find out how long each employee worked before resigning. Afterwards, we can allocated whether they were a short term or long term employee. In tafe_resignations, the column "institute_service" has the amount of time the employee was with the department. However, dete_resignations only includes the employee's start date ("dete_start_date") and their end date ("cease_date"). So let's create a new column that holds the difference.

First off, we'll take a look at the end date:
```python
dete_resignations['cease_date'].value_counts()
```
2012       126  
2013        74  
01/2014     22  
12/2013     17  
06/2013     14  
09/2013     11  
11/2013      9  
07/2013      9  
10/2013      6  
08/2013      4  
05/2013      2  
05/2012      2  
09/2010      1  
07/2006      1  
07/2012      1  
2010         1  
Name: cease_date, dtype: int64  

So we need to remove the month on some of these dates. Unlike before though, we can't replace all of those values with one thing. The below code only keeps the 2nd half (year) after splitting the date. In this situation, we cannot convert to int easily but we can to float:
```python
dete_resignations['cease_date'] = dete_resignations['cease_date'].str.split('/').str[-1]
dete_resignations['cease_date'] = dete_resignations['cease_date'].astype("float")
```

Let's run the value count again:

```python
dete_resignations['cease_date'].value_counts()
```
2013.0    146  
2012.0    129  
2014.0     22  
2010.0      2  
2006.0      1  
Name: cease_date, dtype: int64  

Now that the end date is taken care of, we can look at the start date:
```python
dete_resignations['dete_start_date'].value_counts()
```
2011.0    24  
2008.0    22  
2007.0    21  
2012.0    21  
2010.0    17  
2005.0    15  
2004.0    14  
2009.0    13  
2006.0    13  
2013.0    10  
2000.0     9  
1999.0     8  
1996.0     6  
2002.0     6  
1992.0     6  
1998.0     6  
2003.0     6  
1994.0     6  
1993.0     5  
1990.0     5  
1980.0     5  
1997.0     5  
1991.0     4  
1989.0     4  
1988.0     4  
1995.0     4  
2001.0     3  
1985.0     3  
1986.0     3  
1983.0     2  
1976.0     2  
1974.0     2  
1971.0     1  
1972.0     1  
1984.0     1  
1982.0     1  
1987.0     1  
1975.0     1  
1973.0     1  
1977.0     1  
1963.0     1  
Name: dete_start_date, dtype: int64  

Start dates look good so we can now create a new column that has the difference. We will name the column "institute_service" to match the column with the same name in tafe_resignations.

```python
dete_resignations['institute_service'] = dete_resignations['cease_date'] - dete_resignations['dete_start_date']

print(dete_resignations['institute_service'].head())
```
3      7.0  
5     18.0  
8      3.0  
9     15.0  
11     3.0  
Name: institute_service, dtype: float64  

