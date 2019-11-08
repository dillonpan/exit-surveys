# exit-surveys
Project using the Python package "pandas" and various functions to combine, clean, and analyze data in multiple CSV files  

# Project Details:
For this project, we will take a look at employee exit surveys from two departments of the Queensland government in Australia:  
- Department of Education, Training and Employment (DETE)  
- Technical and Further Education (TAFE)  

Our main objective is to answer the following questions:
- Are employees who only worked for the institutes for a short period of time resigning due to some kind of dissatisfaction?  
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

