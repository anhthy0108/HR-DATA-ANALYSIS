# HR DATA ANALYSIS - SQL / POWER BI
  This project delves into the field of data analysis using SQL and Power BI to uncover important information about HR that can greatly benefit the company. With eye-catching Dashboards, this project provides essential HR metrics such as employee turnover, diversity, recruitment efficiency and performance reviews. This information helps HR professionals make informed decisions and plan strategically for the workforce.

## 1. Project Objectives 
  This project aims to address important questions and issues in human resource management through data analysis. Specifically, the project focuses on understanding and providing solutions for the following issues:
- Employee turnover: What factors influence employees to leave a company and how can it be reduced?
- Diversification: What is the current situation of diversification in the company and what measures can be taken to improve it?
- Recruitment efficiency: Is the recruitment process effective and how can it be optimized?
- Performance appraisal: How is employee performance evaluated and how can overall performance be improved?

## 2. Data Sources
  The data source includes 22,214 personnel records from 2000 to 2020. This data is included in the project's archive.
- Data table structure:
-- **id**: Employee identifier
-- **first_name**: Employee's first name
-- **last_name**: Employee's last name
-- **birthdate**: Employee's date of birth
-- **gender**: Gender of the employee
-- **race**: Race of the employee
-- **department**: The employee's work department
-- **jobtitle**: Employee's job title
-- **location**: Employee's working location
-- **hire_date**: Employee's start date
-- **termdate**: Employee's end date (if any)
-- **location_city**: City where the employee works
-- **location_state**: State where the employee works

## 3. Exploratory Data Analysis
### 3.1. Data cleaning
  Data cleaning and analysis performed on SQL Server involves:
- Data loading & inspection
- Handling missing values
- Data cleaning and analysis
**The steps taken are as follows:**
#### *a/ Create Database*
	CREATE DATABASE human resources;
*b/ Import Data to SQL Server*
- Right-click on Human_Resources > Tasks > Import Data
- Use import wizard to import hr_data.csv to hr_data table
- Verify that the import worked:
#
	USE human_resources;
	SELECT * FROM hr_data;
*c/ Data Cleaning*
The termdate was imported as nvarchar(50). This column contains termination dates, hence it needs to be converted to the date format.
![image](https://github.com/anhthy0108/HR-DATA-ANALYSIS/assets/149523570/0f078356-8214-496e-bef9-1bf9405a092c)

**Update termdate date/time to date:**
- Convert dates to yyyy-MM-dd
#
	UPDATE hr_data
	SET termdate = FORMAT(CONVERT(DATETIME, LEFT(termdate,19),120), 'yyyy-MM-dd');
- Create new column new_termdate
#
	ALTER TABLE hr_data
	ADD new_termdate DATE;
- Copy converted time values from termdate to new_termdate
#
	UPDATE hr_data
	SET new_termdate = CASE WHEN termdate IS NOT NULL AND ISDATE(termdate) = 1 THEN CAST(termdate AS DATETIME)
		ELSE NULL END;
- Create new column "age"
#
	ALTER TABLE hr_data
	ADD age NVARCHAR(50);
- Populate new column with age
#
	UPDATE hr_data
	SET age = DATEDIFF(YEAR,birthdate,GETDATE());
### 3.2. Data analysis
QUESTIONS TO ANSWER FROM THE DATA
**1) What's the age distribution in the company? Age group: (21 to 30; 31 to 40; 41 to 50; 51+)**
*+ Age distribution*
#
	SELECT MAX(age) AS oldest, MIN(age) AS youngest
	FROM hr_data;

*+ Age-group count*
#
	SELECT age_group, count(1) AS count
	FROM(
		SELECT CASE WHEN age >= 21 AND age <= 30 THEN '21 to 30'
					WHEN age >= 31 AND age <= 40 THEN '31 to 40'
					WHEN age >= 41 AND age <= 50 THEN '41 to 50'
					ELSE '51+' 
				END AS age_group
		FROM hr_data
		WHERE new_termdate IS NULL 
	) AS subquery 
	GROUP BY age_group
	ORDER BY age_group;

*+ Age-group by gender*
#
	SELECT age_group, gender, COUNT(1) AS count
	FROM (
		SELECT CASE WHEN age >= 21 AND age <= 30 THEN '21 to 30'
					WHEN age >= 31 AND age <= 40 THEN '31 to 40'
					WHEN age >= 41 AND age <= 50 THEN '41 to 50'
					ELSE '51+' 
				END AS age_group,
				gender
		FROM hr_data
		WHERE new_termdate IS NULL 
	) AS subquery
	GROUP BY age_group, gender
	ORDER BY age_group, gender;

**2) What's the gender breakdown in the company?**
#
	SELECT gender, COUNT(1) AS count
	FROM hr_data
	WHERE new_termdate IS NULL 
	GROUP BY gender
	ORDER BY gender;

**3) How does gender vary across departments and job titles?**
*+ Departments*
#
	SELECT department, gender, COUNT(1) AS count
	FROM hr_data
	WHERE new_termdate IS NULL
	GROUP BY department, gender
	ORDER BY department, gender;

*+ Job titles*
#
	SELECT department, jobtitle, gender, COUNT(1) AS count
	FROM hr_data
	WHERE new_termdate IS NULL
	GROUP BY department, jobtitle, gender
	ORDER BY department, jobtitle, gender;

**4) What's the race distribution in the company?**
#
	SELECT race, COUNT(1) AS count
	FROM hr_data
	WHERE new_termdate IS NULL
	GROUP BY race
	ORDER BY count DESC;

**5) What's the average length of employment in the company?**
#
	SELECT AVG(DATEDIFF(YEAR,hire_date,new_termdate)) AS tenure
	FROM hr_data
	WHERE new_termdate IS NOT NULL AND new_termdate <= GETDATE();

**6) Which department has the highest turnover rate?**
- get total count
- get terminated count
- terminated count/total count
#
	SELECT department, total_count AS 'total count' , terminated_count AS 'terminated count',
		ROUND(CAST(terminated_count AS FLOAT) / total_count,2) * 100 AS turnover_rate 
	FROM (
		SELECT department, COUNT(1) AS total_count,
			SUM(CASE WHEN new_termdate IS NOT NULL AND new_termdate <= GETDATE() THEN 1 ELSE 0 END) AS terminated_count
		FROM hr_data
		GROUP BY department
	) AS subquery
	ORDER BY turnover_rate DESC;

**7) What is the tenure distribution for each department?**
#
	SELECT department, AVG(DATEDIFF(YEAR,hire_date,new_termdate)) AS tenure
	FROM hr_data
	WHERE new_termdate IS NOT NULL AND new_termdate <= GETDATE()
	GROUP BY department
	ORDER BY tenure DESC;

**8) How many employees work remotely for each department?**
#
	SELECT location, department, COUNT(1) AS count
	FROM hr_data
	WHERE new_termdate IS NULL AND location = 'Remote'
	GROUP BY location, department
	ORDER BY count DESC;

	SELECT location, COUNT(1) AS count
	FROM hr_data
	WHERE new_termdate IS NULL
	GROUP BY location
	ORDER BY count DESC;

**9) What's the distribution of employees across different states?**
#
	SELECT location_state, COUNT(1) AS count
	FROM hr_data
	WHERE new_termdate IS NULL 
	GROUP BY location_state
	ORDER BY count DESC;

**10) How are job titles distributed in the company?**
#
	SELECT jobtitle, COUNT(1) AS count
	FROM hr_data
	WHERE new_termdate IS NULL 
	GROUP BY jobtitle
	ORDER BY count DESC;

**11) How have employee hire counts varied over time?**
- calculate hires
- calculate terminations
- (hires-terminations) / hires percent hire change
#
	SELECT hire_year, hires, terminations,
			hires - terminations AS net_change,
			(round(CAST(hires-terminations AS FLOAT)/hires, 2)) * 100 AS percent_hire_change
	FROM (
		SELECT 
		 YEAR(hire_date) AS hire_year,
		 count(*) AS hires,
		 SUM(CASE WHEN new_termdate is not null and new_termdate <= GETDATE() THEN 1 ELSE 0 END) AS terminations
		FROM hr_data
		GROUP BY YEAR(hire_date)
	) AS subquery
	ORDER BY percent_hire_change DESC;

## 4. Data Visualization 
Data visualization is performed mainly on Power BI Desktop.
![image](https://github.com/anhthy0108/HR-DATA-ANALYSIS/assets/149523570/36cb18e2-2a32-47b5-97db-993489c22d3d)
![image](https://github.com/anhthy0108/HR-DATA-ANALYSIS/assets/149523570/aee73e04-232e-4249-961c-6447c118ed3e)

## 5. ConclusionThere are more male employees than female or non-conforming employees.
- The genders are fairly evenly distributed across departments. There are slightly more male employees overall.
- Employees 51+ years old are the fewest in the company. Most employees are 31- 40 years old. 
- Caucasian employees are the majority in the company, followed by mixed race, black, Asian, Hispanic, and native Americans.
- The average length of employment is 7 years.
- Auditing has the highest turnover rate, followed by Legal, Engineering, Human Resources, Research & Development and Training. Business Development & Marketing have the lowest turnover rates.
- Employees tend to stay with the company for 6 - 8 years. Tenure is quite evenly distributed across departments.
- About 25% of employees work remotely. Among them, Engineering has the highest number of remote employees (1360), followed by Accounting (714) and Auditing has the lowest number of remote employees (8).
- Most employees are in Ohio (14,788) followed distantly by Pennsylvania (930) and Illinois (730), Indiana (572), Michigan (569), Kentucky (375) and Wisconsin (321).
- There are 182 job titles in the company, with Research Assistant II taking most of the employees (634) and Assistant Professor, Marketing Manager, Office Assistant IV, Associate Professor and VP of Training and Development taking the just 1 employee each.
- Employee hire counts have increased over the years.
