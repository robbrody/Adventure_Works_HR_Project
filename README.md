# Overview 
## Introduction
In this project, I set out to analyze a custom Human Resources dataset derived from the Adventure Works 2022 dataset in Microsoft SQL Server. This involved creating a comprehensive SQL view that integrated data from 11 different tables, resulting in a robust dataset comprising 31 columns related to employee information. By importing this dataset into Python, I was able to clean and preprocess the data effectively, eliminating duplicate employee entries and ensuring accuracy in capturing the most recent department and pay information.

Utilizing this refined dataset, I conducted in-depth analyses to explore various employee attributes, including gender, marital status, department, pay frequency, shift, organizational level, salary, and age. The culmination of my efforts was the development of a dynamic dashboard featuring five visually informative charts that present key insights by department within the Adventure Works employee dataset. This dashboard not only highlights critical HR metrics but also serves as a powerful tool for understanding employee demographics and organizational dynamics.

## The Questions

Below are the questions I looked to answer in this project:

1. What are the primary demographic and demographic trends within the Adventure Works workforce?

2. How do the departments at Adventure Works differ in terms of employee composition and characteristics?

3. Are there any opportunities to optimize Adventure Works's hiring practices based on the data?

4. What potential enhancements could maximize the dashboard's value for decision-making?

## Tools Used for the Analysis

I used the following tools for this analysis:

* **Python**: The main tool for the analysis, used to analyze the data, and find the answer to the questions. The following libraries were used in Python:
    * **Pandas Library**: Used to analyze the data.
    * **Matplotlib Library**: To visualize the data.
    * **Seaborn Library**: Create more advanced visualizations.
    * **Datetime**: Used to format dates.
    * **Funcformatter**: To format text for plots.
    * **Pyodbc**: Used to load data from SQL server.
    * **Panel**: To create dashboards in Python.
* **SQL**: A program used to manage, manipulate, and query data.
* **Microsoft SQL Server**: A relational database management system.
* **Jupyter Notebooks**: A tool for running Python scripts and easily including notes and analysis.
* **Visual Studio Code**: A program for executing Python scripts.
* **Git & GitHub**: For version control and sharing Python code and analysis.

## Data Collection and View Creation
I reviewed the Adventure Works 2022 dataset in Microsoft SQL Server to identify the necessary tables and columns for this analysis. From this, I created a new view specifically designed to provide HR employee information.

```SQL
Create View HumanResources.vEmployeeHRProject AS
(
	SELECT 
	edh.BusinessEntityID,
	edh.DepartmentID,
	edh.ShiftID,
	edh.StartDate, 
	edh.EndDate,
	e.JobTitle,
	e.BirthDate,
	e.MaritalStatus,
	e.Gender,
	e.HireDate,
	e.SalariedFlag,
	e.VacationHours,
	e.SickLeaveHours,
	e.OrganizationLevel,
	p.PersonType,
	p.FirstName,
	p.LastName,
	d.Name DepartmentName, 
	d.GroupName DepartmentGroup,
	eph.RateChangeDate,
	eph.Rate,
	eph.PayFrequency,
	s.Name ShiftName,
	s.StartTime ShiftStart,
	s.EndTime ShiftEnd,
	bea.AddressID,
	a.City,
	a.PostalCode,
	pp.PhoneNumber,
	pp.PhoneNumberTypeID,
	pnt.Name PhoneType

	FROM HumanResources.EmployeeDepartmentHistory edh
	LEFT JOIN HumanResources.Employee e
	ON e.BusinessEntityID = edh.BusinessEntityID
	LEFT JOIN Person.Person p
	ON p.BusinessEntityID = edh.BusinessEntityID
	LEFT JOIN HumanResources.Department d
	ON d.DepartmentID = edh.DepartmentID
	LEFT JOIN HumanResources.EmployeePayHistory eph
	ON eph.BusinessEntityID = edh.BusinessEntityID
	LEFT JOIN HumanResources.Shift s
	ON s.ShiftID = edh.ShiftID
	LEFT JOIN Person.BusinessEntityAddress bea
	ON bea.BusinessEntityID = edh.BusinessEntityID
	LEFT JOIN Person.Address a
	ON a.AddressID = bea.AddressID
	LEFT JOIN Person.PersonPhone pp
	ON pp.BusinessEntityID = edh.BusinessEntityID
	LEFT JOIN Person.PhoneNumberType pnt
	ON pnt.PhoneNumberTypeID = pp.PhoneNumberTypeID
)

```

## Load Libraries and Dataset via SQL
I loaded all essential libraries required to construct the dashboard, including tools for data visualization, loading the data and text generation. The dataset, initially created in Microsoft SQL Server, was then imported into Python to facilitate the analysis and final dashboard creation.

``` python
import pyodbc
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.ticker as mticker
import seaborn as sns
from datetime import datetime 
from matplotlib.ticker import FuncFormatter

# Define the connection string
conn = pyodbc.connect(
    'DRIVER={ODBC Driver 17 for SQL Server};'
    'SERVER=localhost\\SQLEXPRESS;'
    'DATABASE=AdventureWorks2022;'
    'Trusted_Connection=yes;'
)


# Write the SQL query to select data from the view
sql_query = "SELECT * FROM HumanResources.vEmployeeHRProject ORDER BY BusinessEntityID"

# Load the data from SQL Server into a pandas DataFrame
df = pd.read_sql(sql_query, conn)

# Display the first few rows of the DataFrame
df.head()

# Close the connection
conn.close()

```

## Data Cleaning
The data was refined by removing duplicate employee entries associated with outdated departments and pay rates. Only the most recent information was retained for each employee, ensuring accuracy and relevance in the dataset.

``` python
# Step 1: Filter for rows where 'EndDate' is null
filtered_df = df[df['EndDate'].isnull()]

# Step 2: Sort by 'RateChangeDate' within each 'BusinessEntityID'
# and keep only the latest one
cleaned_df = (filtered_df
              .sort_values('RateChangeDate', ascending=False)
              .groupby('BusinessEntityID')
              .first()  # Keep the first entry after sorting (latest RateChangeDate)
              .reset_index())

# Fill NaN values in 'OrganizationLevel' with 0
cleaned_df['OrganizationLevel'] = cleaned_df['OrganizationLevel'].fillna(0)

```
## Adding Age and Salary Columns to the Dataset
New columns for Age and Salary (calculated based on 2,080 hours at the current pay rate) were created to support a Salary vs. Age scatterplot for the dashboard. These additions provide essential data points for the analysis.

``` python
# Convert 'BirthDate' to datetime
cleaned_df['BirthDate'] = pd.to_datetime(cleaned_df['BirthDate'])

# Ensure 'Rate' is a string before removing the dollar sign and converting to numeric
cleaned_df['Rate'] = cleaned_df['Rate'].astype(str).str.replace('$', '').str.strip()
cleaned_df['Rate'] = pd.to_numeric(cleaned_df['Rate'], errors='coerce')

# Calculate Age
current_date = datetime.now()
cleaned_df['Age'] = (current_date - cleaned_df['BirthDate']).dt.days // 365

# Calculate Salary based on hourly rate (assuming 2080 working hours in a year)
cleaned_df['Salary'] = cleaned_df['Rate'] * 2080  # Annual salary

```

