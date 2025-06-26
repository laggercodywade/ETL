# ETL Pipeline for Data Science Hiring Prediction

## Project Description
This project practices ETL (Extract, Transform, Load) processes to integrate heterogeneous data sources related to candidates enrolling in a Data Science training program. The goal is to create a unified dataset for analysis.

## Data Sources
| Source Type          | Format       | Link/Connection Details                                                                 | Key Fields                              |
|----------------------|--------------|-----------------------------------------------------------------------------------------|-----------------------------------------|
| Enrollies' Data      | Google Sheets| [Link](https://docs.google.com/spreadsheets/d/1VCkHwBjJGRJ21asd9pxW4_0z2PWuKhbLR3gUHm-p4GI/edit?usp=sharing) | `enrollee_id`, `city`, `gender`         |
| Education Records    | Excel        | [Download](https://assets.swisscoding.edu.vn/company_course/enrollies_education.xlsx)    | `enrollee_id`, `education_level`, `major_discipline` |
| Work Experience      | CSV          | [Download](https://assets.swisscoding.edu.vn/company_course/work_experience.csv)         | `enrollee_id`, `experience`, `company_type` |
| Training Hours       | MySQL        | Host: `112.213.86.31:3360`<br>User: `etl_practice`<br>Password: `550814`<br>DB: `company_course` | `enrollee_id`, `training_hours`         |
| City Development Index | HTML Table  | [Website](https://sca-programming-school.github.io/city_development_index/index.html)    | `city`, `development_index`             |
| Employment Status    | MySQL        | Same as Training Hours                                                                   | `enrollee_id`, `is_employed` (target)   |

---

## ETL Steps

### 1. Extraction
```python
# Example: Extract from Google Sheets
link = 'https://docs.google.com/spreadsheets/d/1VCkHwBjJGRJ21asd9pxW4_0z2PWuKhbLR3gUHm-p4GI/edit?usp=sharing'
google_sheet_id = '1VCkHwBjJGRJ21asd9pxW4_0z2PWuKhbLR3gUHm-p4GI'
url= 'https://docs.google.com/spreadsheets/d/' + google_sheet_id + '/export?format=xlsx'
enrollies_df = pd.read_excel(url,sheet_name='enrollies')

# Example: Extract from XLXS ( Excel )
enrollies_edu_df = pd.read_excel('/content/drive/MyDrive/Colab Notebooks/ETL/enrollies_education.xlsx')

# Example: Extract from CSV
working_experience_df = pd.read_csv('/content/drive/MyDrive/Colab Notebooks/ETL/work_experience.csv')

# Example: Extract from website
page_url = 'https://sca-programming-school.github.io/city_development_index/index.html'
city_development_index_df = pd.read_html(page_url)[0]

# Example: Extract from MySQL
!pip install pymysql
from sqlalchemy import create_engine
db_url= 'mysql+pymysql://etl_practice:550814@112.213.86.31:3360/company_course'
engine = create_engine(db_url)
trainning_hour_df = pd.read_sql_table('training_hours',engine)
```
### 2. Transformation
#### 2.1 Enrollies' data
``` python
# Enrollies' data
enrollies_df.head()
#Fixing data type
enrollies_df['full_name'] = enrollies_df['full_name'].astype('string')
enrollies_df['city'] = enrollies_df['city'].astype('string')
#Missing data handling
enrollies_df['gender'] = enrollies_df['gender'].fillna(enrollies_df['gender'].mode()[0])
## gender -> Category
enrollies_df['gender'] = enrollies_df['gender'].astype('category')
enrollies_df.info()

2.2 Enrollies education
# Enrollies education
enrollies_edu_df.info()

# A large amount of missing data so we will fillna with 'Unknow'
enrollies_edu_df['enrollied_uninversity'] = enrollies_edu_df['enrolled_university'].fillna('Unknow')
enrollies_edu_df['education_level'] = enrollies_edu_df['education_level'].fillna('Unknow')
enrollies_edu_df['major_discipline'] = enrollies_edu_df['major_discipline'].fillna('Unknow')

# With a lots of same values type in columns we convert them to category type to save memory
cat_cols= ['enrollied_uninversity','education_level','major_discipline','enrolled_university']
enrollies_edu_df[cat_cols] = enrollies_edu_df[cat_cols].astype('category')

2.3 Enrollies working 
# Enrollies working
working_experience_df.info()

# We will do the same with Enrollies education table
experience_mode = working_experience_df['experience'].mode()[0]
working_experience_df['experience'] = working_experience_df['experience'].fillna(experience_mode)
working_experience_df['company_size'] = working_experience_df['company_size'].fillna('Unknow')
working_experience_df['company_type'] = working_experience_df['company_type'].fillna('Unknow')
working_experience_df['last_new_job'] = working_experience_df['last_new_job'].fillna('Unknow')

# Fixing data type
cat_cols = ['relevent_experience','company_size','company_type','last_new_job']
working_experience_df[cat_cols] = working_experience_df[cat_cols].astype('category')
```
#### 3. Load
``` python
db = 'data_warehouse.db'
engine = create_engine('sqlite:///data_warehouse.db')

employment_df.to_sql('Fact_Employment',engine,if_exists='replace',index=False)
city_development_index_df.to_sql('Dim_City',engine,if_exists='replace',index=False)
trainning_hour_df.to_sql('Dim_Training_Hours',engine,if_exists='replace',index=False)
working_experience_df.to_sql('Dim_Working_Experience',engine,if_exists='replace',index=False)
enrollies_edu_df.to_sql('Dim_Enrollies_Education',engine,if_exists='replace',index=False)
enrollies_df.to_sql('Dim_Enrollies',engine,if_exists='replace',index=False)
```
