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

Start date looks good so we can now create a new column that has the difference. We will name the column "institute_service" to match the column with the same name in tafe_resignations.

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

Now that we have our finished new column, we can move on to the tafe_resignations dataset.

```python
print(tafe_resignations['institute_service'].value_counts(dropna=False))
```
Less than 1 year      73  
1-2                   64  
3-4                   63  
NaN                   50  
5-6                   33  
11-20                 26  
7-10                  21  
More than 20 years    10  
Name: institute_service, dtype: int64  

Looks like the main issue is that the service times listed are in some form of a range. To fix this, we will filter out and take the lower number of the range or the only number in the case of "less than 1" and "more than 20". Losing the higher number of the range won't make much of a difference when we categorize the numbers later on.  

Unlike before, we can't just split or overwrite the values with the same value. This time, we'll use regex to grab the first digit(s) that comes up in each value. Then we will turn it in to a string value, matching the "institute_service" column in dete_resignations:
```python
tafe_resignations['institute_service'] = tafe_resignations['institute_service'].astype('str').str.extract(r'(\d+)')
tafe_resignations['institute_service'] = tafe_resignations['institute_service'].astype('float')

print(tafe_resignations['institute_service'].value_counts(dropna=False))
```
1.0     137
3.0      63
NaN      50
5.0      33
11.0     26
7.0      21
20.0     10
Name: institute_service, dtype: int64

# Part3: Identifying Dissatisfied Employees
For both datasets, there's multiple columns with different disstisfaction reasons. The values within those columns are pretty much a yes/no type of response. Looking over all the column headers for both datasets, it looks the the following are the important ones to identify dissatisfaction:

tafe_survey_updated:  
- Contributing Factors. Dissatisfaction  
- Contributing Factors. Job Dissatisfaction  

dafe_survey_updated:  
- job_dissatisfaction  
- dissatisfaction_with_the_department  
- physical_work_environment  
- lack_of_recognition  
- lack_of_job_security  
- work_location  
- employment_conditions  
- work_life_balance  
- workload  

We can use value_counts() on each of these headers to see what values are in them. I've gone ahead and done that. Looks like the results for each column in their respective dataset resulted in the exact same values.  

Thus, I will just provide two columns below as examples:
```python
print(tafe_resignations['Contributing Factors. Dissatisfaction'].value_counts(dropna=False))

print(dete_resignations['job_dissatisfaction'].value_counts(dropna=False))
```

(Note: Actual Dash) -                      277  
Contributing Factors. Dissatisfaction      55  
NaN                                         8  
Name: Contributing Factors. Dissatisfaction, dtype: int64  


False    270  
True      41  
Name: job_dissatisfaction, dtype: int64  

In job_dissatisfaction, we have a bunch of columns with true/false listed as values. We can go row by row and see if any of the columns listed above contain "True" as a value for that row. As long as one does for that row, it confirms the employee resigned due to some form of dissatisfaction. We can create a new column that contains a final "True" value, or "False" if none of the columns had a "True" value.

We can do the same with tafe_resignations, we just need to first conert the current values to either "True" or "False". Let's go ahead and do that. We can also create the new column ("dissatisfied") and include the final True/False value in it under the same code:
```python
def update_vals(x):
    if x == '-':
        return False
    elif pandas.isnull(x):
        return numpy.nan
    else:
        return True


# When going through each row, we run both columns through the update_vals() function.
# If either value comes back "True", we send "True" to the new column
tafe_resignations['dissatisfied'] = tafe_resignations[['Contributing Factors. Dissatisfaction', 'Contributing Factors. Job Dissatisfaction']].applymap(update_vals).any(axis=1, skipna=False)

tafe_resignations_up = tafe_resignations.copy()
print(tafe_resignations_up['dissatisfied'].value_counts(dropna=False))
```
False    241  
True      91  
NaN        8  
Name: dissatisfied, dtype: int64  

Now we can also use the .any() function on dete_resignations and create the same new column, "dissatisfied":
```python
dete_resignations['dissatisfied'] = dete_resignations[['job_dissatisfaction',
       'dissatisfaction_with_the_department', 'physical_work_environment',
       'lack_of_recognition', 'lack_of_job_security', 'work_location',
       'employment_conditions', 'work_life_balance',
       'workload']].any(1, skipna=False)
       
dete_resignations_up = dete_resignations.copy()
print(dete_resignations_up['dissatisfied'].value_counts(dropna=False))
```

False    162  
True     149  
Name: dissatisfied, dtype: int64  

# Combining the Data
Before we actually combine the data, we're going to create another column that will signify which previous dataset the original row came from.
```python
dete_resignations_up['institute'] = 'DETE'
tafe_resignations_up['institute'] = 'TAFE'
```

Now let's combine the datasets in to one called "combined"
```python
combined = pandas.concat([dete_resignations_up, tafe_resignations_up], ignore_index=True, sort=True)
```
When combing the data, we're going to hae many columns with a lot of empty/null values. We can drop the columns where there not enough values that are "non-null":
```python
# optional argument thresh means the column/row needs at least 500 "non-null" for the column not to get dropped.
combined_updated = combined.dropna(thresh=500, axis=1).copy()
```

# Categorizing as Short Term or Long Term
There is no official definition on what constitutes short term vs long term, so we're going to use the following guidelines taken from [this article](https://www.businesswire.com/news/home/20171108006002/en/Age-Number-Engage-Employees-Career-Stage). This article includes some details on how career stages may affect employees needs and thus satisfaction. This could assist in detailing what changes the departments should consider to improve employee morale. The article breaks down career stages to the following values:  

- New: Less than 3 years in the workplace  
- Experienced: 3-6 years in the workplace  
- Established: 7-10 years in the workplace  
- Veteran: 11 or more years in the workplace  

So we now have a combined database, a column filled with True/False values that confirm whether or not the ex-employee resigned due to some formm of dissatisfaction, and a column with an approximation of how long he employee was with the department (in years). 

Lets take a look at the :
```python
def transform_service(val):
    if val >= 11:
        return "Veteran"
    elif 7 <= val < 11:
        return "Established"
    elif 3 <= val < 7:
        return "Experienced"
    elif pandas.isnull(val):
        return numpy.nan
    else:
        return "New"


combined_updated['service_cat'] = combined_updated['institute_service'].apply(transform_service)

print(combined_updated['service_cat'].value_counts())
```

New            193  
Experienced    172  
Veteran        136  
Established     62  
Name: service_cat, dtype: int64  

# Analysis
Now that we have consolidated all the information we need, we can analyze each "service_cat" and check the percentage of how many in each group resigned due to dissatisfaction.

```python
print(combined_updated.loc[combined_updated['service_cat'] == 'Established', 'dissatisfied'].value_counts(normalize=True))
```
True     0.516129  
False    0.483871  
Name: dissatisfied, dtype: float64  

We could do the above for each the other "service_cat" or we can do it all at once using a Pivot Table. The only issue with uing a Pivot Table is that the values we're using ("dissatisfied") includes NaN/Null values. Pivot tables read boolean values as True=1 and False=0, NaN values would cause an error.
```python
combined_updated['dissatisfied'].value_counts(dropna=False)
```
False    403  
True     240  
NaN        8  
Name: dissatisfied, dtype: int64  

Since only 8 rows have a NaN/Null value in the "dissatisfied" column, we can just overwrite them to the more common response, False:
```python
combined_updated['dissatisfied'] = combined_updated['dissatisfied'].fillna(False)
```

Now we can create our pivot table:
```python
dis_pct = combined_updated.pivot_table(index='service_cat', values='dissatisfied', aggfunc='mean')

print(dis_pct)
````
_              dissatisfied  
service_cat                
Established      0.516129  
Experienced      0.343023  
New              0.295337  
Veteran          0.485294  

Lastly, we can plot the data to better visualize it:
```python
import matplotlib.pyplot

dis_pct.plot(kind='bar', rot=0)

matplotlib.pyplot.show()
```
![image](https://user-images.githubusercontent.com/57373723/68516268-58d50100-0238-11ea-8482-7385bcf2ba6f.png)

We could have also categorized New & Experienced as Short Term while Veteran & Established as Long Term and redo the Pivot Table. However, the Pivot Table and plot easily tells us that employees who were with either department 7+ years were more likely to be dissatisfied when they resigned.

While the end result shows us the results (resignation), it does not prove anything about causation of their dissatisfaction. It also does not show anything about possible dissatisfaction in current employees. However, this can be used to start research on current Veteran and Established employees to see if they are dissatsified and why before more resign on bad terms.
