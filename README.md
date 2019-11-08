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
print(dete_survey.info())
```

   ID         SeparationType Cease Date  ... South Sea Disability NESB  
0   1  Ill Health Retirement    08/2012  ...       NaN        NaN  Yes  


RangeIndex: 822 entries, 0 to 821  
Data columns (total 56 columns):  
ID                                     822 non-null int64  
SeparationType                         822 non-null object  
Cease Date                             822 non-null object  
DETE Start Date                        822 non-null object  
Role Start Date                        822 non-null object  
Position                               817 non-null object  
Classification                         455 non-null object  
Region                                 822 non-null object  
Business Unit                          126 non-null object  
Employment Status                      817 non-null object  
Career move to public sector           822 non-null bool  
Career move to private sector          822 non-null bool  
Interpersonal conflicts                822 non-null bool  
Job dissatisfaction                    822 non-null bool  
Dissatisfaction with the department    822 non-null bool  
Physical work environment              822 non-null bool  
Lack of recognition                    822 non-null bool  
Lack of job security                   822 non-null bool  
Work location                          822 non-null bool  
Employment conditions                  822 non-null bool  
Maternity/family                       822 non-null bool  
Relocation                             822 non-null bool  
 Study/Travel                           822 non-null bool  
Ill Health                             822 non-null bool  
Traumatic incident                     822 non-null bool  
Work life balance                      822 non-null bool  
Workload                               822 non-null bool  
None of the above                      822 non-null bool  
Professional Development               808 non-null object  
Opportunities for promotion            735 non-null object  
Staff morale                           816 non-null object  
Workplace issue                        788 non-null object  
Physical environment                   817 non-null object  
Worklife balance                       815 non-null object  
Stress and pressure support            810 non-null object  
Performance of supervisor              813 non-null object  
Peer support                           812 non-null object  
Initiative                             813 non-null object  
Skills                                 811 non-null object  
Coach                                  767 non-null object  
Career Aspirations                     746 non-null object  
Feedback                               792 non-null object  
Further PD                             768 non-null object  
Communication                          814 non-null object  
My say                                 812 non-null object  
Information                            816 non-null object  
Kept informed                          813 non-null object  
Wellness programs                      766 non-null object  
Health & Safety                        793 non-null object  
Gender                                 798 non-null object  
Age                                    811 non-null object  
Aboriginal                             16 non-null object  
Torres Strait                          3 non-null object  
South Sea                              7 non-null object  
Disability                             23 non-null object  
NESB                                   32 non-null object  
dtypes: bool(18), int64(1), object(37)  
memory usage: 258.6+ KB  

