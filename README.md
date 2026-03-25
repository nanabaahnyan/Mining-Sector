# Mining Sector Dataset Analysis
## Overview
And i'm here again to upgrade my skills in Data Science by analyzing this dataset. I generated the dataset using python codes and introduced random mistakes in the dataset for data cleaning. This comes with the codes for performing various analysis and images of the results.

## Questions
1. Find the productivity (avg ore extracted per hour worked) by ore type.
2. Role with most Major Accident incidents.
3. Incident rate per 100 shifts.
4. What is the month-year with highest total ore extraction

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