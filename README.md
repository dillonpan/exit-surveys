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
print(dete_survey.head(1))
```

   ID                    SeparationType Cease Date  ... South Sea Disability NESB  
0   1             Ill Health Retirement    08/2012  ...       NaN        NaN  Yes  
1   2  Voluntary Early Retirement (VER)    08/2012  ...       NaN        NaN  NaN  
2   3  Voluntary Early Retirement (VER)    05/2012  ...       NaN        NaN  NaN  

[3 rows x 56 columns]

```python
tafe_survey = pandas.read_csv("tafe_survey.csv")
tafe_survey.head()
```

/           Record ID  ... LengthofServiceCurrent. Length of Service at current workplace (in years)  
0  634133009996094000  ...                                                1-2                         
1  634133654064531000  ...                                                NaN                       
2  634138845606563000  ...                                                NaN                       

[3 rows x 72 columns]

