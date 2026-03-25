# Mining Sector Dataset Analysis
## Overview
And i'm here again to upgrade my skills in Data Science by analyzing this dataset. I generated the dataset using python codes and introduced random mistakes in the dataset for data cleaning. This comes with the codes for performing various analysis and images of the results.

## Questions
1. Find the productivity (avg ore extracted per hour worked) by ore type.
2. Role with most Major Accident incidents.
3. Incident rate per 100 shifts.
4. What is the month-year with highest total ore extraction.

## Setting up and Downloading various libraries
This is where i set up my IDE (Visual Studio code) by downloading all the extensions i will need (Had already installed Visual Studio Code). I then added my codebase to Github repository by using git and always committing changes often.

![Setting up my code base](<images/setting up.png>)
*Setting up my code base*

### Tools used
* Python
* Visual studio code
* Jupyter Notebook
* Git
* Github

### Libraries
* Pandas
* Matplotlib
* Seaborn
* scikit-learn
* Generating Random codes

### Code for generating random mining data
```Python
import pandas as pd
import random
from datetime import datetime, timedelta
import numpy as np

# Lists for fake data
first_names = ['John', 'Jane', 'Alice', 'Bob', 'Charlie', 'David', 'Eve', 'Frank', 'Grace', 'Henry', 'Ivy', 'Jack', 'Katie', 'Leo', 'Mia', 'Noah', 'Olivia', 'Paul', 'Quinn', 'Ryan'] * 25
last_names = ['Smith', 'Doe', 'Johnson', 'Brown', 'Davis', 'Wilson', 'Moore', 'Taylor', 'Anderson', 'Thomas', 'Jackson', 'White', 'Harris', 'Martin', 'Thompson', 'Garcia', 'Martinez', 'Robinson', 'Clark', 'Rodriguez'] * 25
sites = ['Mine A', 'Mine B', 'Mine C', 'Mine D', 'Mine E']
roles = ['Miner', 'Engineer', 'Supervisor', 'Technician', 'Safety Officer']
ore_types = ['Gold', 'Coal', 'Iron', 'Copper', 'Diamond']
incident_types = ['Minor Injury', 'Equipment Failure', 'None', 'Environmental', 'Major Accident']
departments = ['Operations', 'Maintenance', 'Safety', 'Administration', 'Exploration']

# Generate employees
employees = []
for i in range(1, 501):
    fn = random.choice(first_names)
    ln = random.choice(last_names)
    username = f"{fn.lower()}_{ln.lower()}_{random.randint(1,999)}"
    email = f"{username}@miningco.com"
    hire_date = datetime(2015, 1, 1) + timedelta(days=random.randint(0, 3285))  # Up to ~9 years
    age = random.randint(25, 65)
    gender = random.choice(['M', 'F', 'Other'])
    phone = f"({random.randint(100,999)}){random.randint(100,999)}-{random.randint(1000,9999)}"
    site = random.choice(sites)
    role = random.choice(roles)
    department = random.choice(departments)
    employees.append({
        'EmployeeID': i,
        'Username': username,
        'FirstName': fn,
        'LastName': ln,
        'Email': email,
        'HireDate': hire_date.strftime('%Y-%m-%d'),
        'Age': age,
        'Gender': gender,
        'Phone': phone,
        'Site': site,
        'Role': role,
        'Department': department
    })

# Generate shift histories
rows = []
shift_id = 1
for emp in employees:
    num_shifts = random.randint(0, 10)
    if num_shifts == 0:
        row = emp.copy()
        row.update({
            'ShiftID': np.nan,
            'ShiftDate': np.nan,
            'HoursWorked': np.nan,
            'OreExtracted': np.nan,
            'OreType': np.nan,
            'Incident': np.nan,
            'SafetyScore': np.nan
        })
        rows.append(row)
    else:
        for _ in range(num_shifts):
            shift_date_dt = datetime.strptime(emp['HireDate'], '%Y-%m-%d') + timedelta(days=random.randint(1, 730))
            shift_date = shift_date_dt.strftime('%Y-%m-%d')
            hours = random.randint(4, 12)
            ore_ext = round(random.uniform(0, 1000), 2) if random.random() > 0.2 else 0
            ore_type = random.choice(ore_types) if ore_ext > 0 else np.nan
            incident = random.choice(incident_types)
            safety = random.randint(1, 100)
            row = emp.copy()
            row.update({
                'ShiftID': shift_id,
                'ShiftDate': shift_date,
                'HoursWorked': hours,
                'OreExtracted': ore_ext,
                'OreType': ore_type,
                'Incident': incident,
                'SafetyScore': safety
            })
            rows.append(row)
            shift_id += 1

# Create DataFrame
df_mining = pd.DataFrame(rows)

# Introduce mistakes (dirtiness)
for col in df_mining.columns:
    if col not in ['EmployeeID']:
        mask = [random.random() < 0.03 for _ in range(len(df_mining))]
        df_mining.loc[mask, col] = np.nan

# Typos in names
for i in range(len(df_mining)):
    if random.random() < 0.05 and not pd.isna(df_mining.loc[i, 'FirstName']):
        df_mining.loc[i, 'FirstName'] += random.choice(['x', '!', '123', ' '])
    if random.random() < 0.05 and not pd.isna(df_mining.loc[i, 'LastName']):
        df_mining.loc[i, 'LastName'] += 'typo'

# Invalid emails
for i in range(len(df_mining)):
    if random.random() < 0.03 and not pd.isna(df_mining.loc[i, 'Email']):
        if random.random() < 0.5:
            df_mining.loc[i, 'Email'] = df_mining.loc[i, 'Email'].replace('@', '')
        else:
            df_mining.loc[i, 'Email'] = 'invalid_email'

# Negative or unrealistic ages
for i in range(len(df_mining)):
    if random.random() < 0.02 and not pd.isna(df_mining.loc[i, 'Age']):
        if random.random() < 0.5:
            df_mining.loc[i, 'Age'] = -int(df_mining.loc[i, 'Age'])
        else:
            df_mining.loc[i, 'Age'] = random.randint(100, 200)

# Shift dates before hire
for i in range(len(df_mining)):
    if random.random() < 0.02 and not pd.isna(df_mining.loc[i, 'ShiftDate']) and not pd.isna(df_mining.loc[i, 'HireDate']):
        hire_dt = datetime.strptime(df_mining.loc[i, 'HireDate'], '%Y-%m-%d')
        shift_dt = hire_dt - timedelta(days=random.randint(1, 365))
        df_mining.loc[i, 'ShiftDate'] = shift_dt.strftime('%Y-%m-%d')

# Negative hours or ore
for i in range(len(df_mining)):
    if random.random() < 0.01 and not pd.isna(df_mining.loc[i, 'HoursWorked']):
        df_mining.loc[i, 'HoursWorked'] = -int(df_mining.loc[i, 'HoursWorked'])
    if random.random() < 0.01 and not pd.isna(df_mining.loc[i, 'OreExtracted']):
        df_mining.loc[i, 'OreExtracted'] = -float(df_mining.loc[i, 'OreExtracted'])

# Invalid safety scores
for i in range(len(df_mining)):
    if random.random() < 0.03 and not pd.isna(df_mining.loc[i, 'SafetyScore']):
        df_mining.loc[i, 'SafetyScore'] = random.choice([-10, 110])

# Save to csv
df_mining.to_csv('mining_company_data.csv', index=False)
print("Mining company csv file 'mining_company_data.csv' generated successfully!")
```

## Analyzing the dataset using the questions
Each Jupiter notebook in this project is aimed to answer a specific question in the questions listed at the top.

### 1.  Find the productivity (avg ore extracted per hour worked) by ore type.
To solve for the productivity by ore type, i imported all my libraries and also my dataset. I then cleaned the dates by converting it from strings to dateTime. I have to then extract the dataset based on OreExtracted and HoursWorked if only they are greater than 0. I then find the mean by using the groupBy function and also sort them in descending order.

View my detailed code snippets here: [Productivity by ore](productivity_by_ore/productivity_by_ore.ipynb)

![Visualizing solution by graph](<images/Productivity by ore (Visualization).png>)

### 2. Role with most Major Accident incidents.
Accidents occur much in mining companies, as part of analyzing this dataset, i'm tasked to find the most accident incidents by role. I had to import my libraries and do some data cleaning as usual. Now back tto the main problem, to bring out the solution, i extracted accidents flagged as Major Accident and then matched the roles against those major accidents and performed value count on it get the total number of incidents for each role.

View my detailed code snippets here: [Roles with major accident incidents](roles_with_major_accidents/roles_with_major_accidents.ipynb)

![Visualization of roles with major accidents](<images/Roles wit major accidents (Visualization).png>)
*The role with the most accident incidents is the supervisory role that recorded 104 major incidents*

### 3. Incident rate per 100 shifts.
Incidents rate per 100 shift was analyzed by filtering the Incident column by removing NaN and None values. I'd already imported all the necessary libraries and my dataset and also done data cleaning. I then extracted the length of the total Incidents as total_shifts and incident_shifts as Incident_Count. With these two length, i divided the total_shifts by incidents_shift and multiplied it by 100 to get the rate.

View my detailed code snippets here: [Incident rate per 100 shifts](<incident rate/incident_rate.ipynb>)

#### Codes
```Python

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import seaborn as sns

df = pd.read_csv("d://Mining Sector//files//mining_company_data.csv")
df["HireDate"]  = pd.to_datetime(df["HireDate"],  errors="coerce")
df["ShiftDate"] = pd.to_datetime(df["ShiftDate"], errors="coerce")
 
sns.set_theme(style="whitegrid", palette="muted")
plt.rcParams.update({"figure.dpi": 130, "axes.titlesize": 13, "axes.labelsize": 11})

incident_shifts = df[df["Incident"].notna() & (df["Incident"].str.strip() != "None")]
total_shifts    = len(df)
incident_count  = len(incident_shifts)
rate_per_100    = incident_count / total_shifts * 100
print(f"Incident rate: {rate_per_100:.2f} per 100 shifts")


```

*Incident shift was 76.30 per 100 shifts*

### 4. What is the month-year with highest total ore extraction.
I analyzed peak ore extraction by finding the ore extracted which is greater than one and also got the associated mining shift date. I then used the sum function to get the sum of each ore mining by month-year and sort them in descending order.

View my detailed code snippets here: [Highest total ore](highest_total_ore/highest_total_ore_extraction.ipynb)

#### Codes
```Python
df_dated = df[df["OreExtracted"] > 0].dropna(subset=["ShiftDate"]).copy()
df_dated["MonthYear"] = df_dated["ShiftDate"].dt.to_period("M")
monthly_ore = df_dated.groupby("MonthYear")["OreExtracted"].sum().sort_values(ascending=False)
print(f"Peak ore extraction month: {monthly_ore.idxmax()} ({monthly_ore.max():,.2f} units)")
```

### Visualization
![Peak ore extraction](<images/Peak ore extraction (Visualization).png>)
*Peak ore extraction month is 2018-01 with 14,097.96 units of ore extraction*

## Conclusion
After performing analysis on this mining sector dataset, i got to interact with newer functions and visualization concepts that has broadened my understanding and knowledge. I got to appreciate how well data can be manipulated to bring about information that aids organizations and businesses to make decisions. I was compelled to do more of these analysis since it becomes more interesting to grasp more whiles solving questions.
This dataset can be acted upon more to bring out some of the best analytical concepts in learning.

*Note*
Ways i used to solve these dataset questions can be different yet attain the same result. All tools and materials that were not cited and acknowledged is indirectly acknowledged and will be added when it comes to mind.

Thanks to **Luke Barousse** for the massive and inspiring tutorial and also thanks to the Python theme for massive libraries. <br>
Check him out on YouTube *https://www.youtube.com/@LukeBarousse*