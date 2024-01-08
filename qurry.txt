create database Employee_attrition;
use Employee_attrition;

USE Employee_attrition;

SELECT *
FROM human_resources;

-- Data cleaning --



EXEC sp_rename 'human_resources.id', 'employee_id', 'COLUMN';

ALTER TABLE human_resources
ALTER COLUMN employee_id VARCHAR(20) NULL;

-- sql cloumn ditels 
--1
SELECT *
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'human_resources';

--2

EXEC sp_columns 'human_resources';



-- change date of birtday formet

UPDATE human_resources
SET birthdate = CASE
    WHEN CHARINDEX('/', birthdate) > 0 THEN CONVERT(VARCHAR, TRY_CAST(birthdate AS DATE), 23)
    WHEN CHARINDEX('-', birthdate) > 0 THEN CONVERT(VARCHAR, TRY_CAST(birthdate AS DATE), 23)
    ELSE NULL
END;


-- change birthdate data type

ALTER TABLE human_resources
ALTER COLUMN birthdate DATE;


-- Alter the table to change the data type of the termdate column

UPDATE human_resources
SET termdate = TRY_CONVERT(DATETIME, termdate, 120)
WHERE termdate IS NOT NULL AND termdate != '' AND TRY_CONVERT(DATETIME, termdate, 120) IS NOT NULL;

-- Update the existing data with error handling
UPDATE human_resources
SET termdate = TRY_CONVERT(DATETIME, termdate, 120)
WHERE termdate IS NOT NULL AND termdate != '';



-- Add a new column 'human_resources' to the 'hr' table
ALTER TABLE human_resources
ADD age INT;

-- Update the 'age' column based on the difference between 'birthdate' and current date

UPDATE human_resources
SET age = DATEDIFF(YEAR, birthdate, GETDATE());



SELECT min(age), max(age) FROM human_resources;


-- 1. What is the gender breakdown of employees in the company
SELECT * FROM human_resources


SELECT gender, COUNT(*) AS count 
FROM human_resources
WHERE termdate IS NULL
GROUP BY gender;

-- or--

SELECT gender, COUNT(*) as count
FROM human_resources
GROUP BY gender;


-- 2. What is the race breakdown of employees in the company
SELECT race , COUNT(*) AS count
FROM human_resources

WHERE termdate IS NULL
GROUP BY race;

-- 3. What is the age distribution of employees in the company

SELECT
  FLOOR(DATEDIFF(DAY, birthdate, GETDATE()) / 365.25) AS age,
  COUNT(*) AS count
FROM
  human_resources
WHERE
  termdate IS NULL
GROUP BY
  FLOOR(DATEDIFF(DAY, birthdate, GETDATE()) / 365.25)
ORDER BY
  age;

  --What is the age distribution of employees in the company(divided age group)
  SELECT 
    CASE
        WHEN FLOOR(DATEDIFF(DAY, birthdate, GETDATE()) / 365.25) BETWEEN 18 AND 24 THEN '18-24'
        WHEN FLOOR(DATEDIFF(DAY, birthdate, GETDATE()) / 365.25) BETWEEN 25 AND 34 THEN '25-34'
        WHEN FLOOR(DATEDIFF(DAY, birthdate, GETDATE()) / 365.25) BETWEEN 35 AND 44 THEN '35-44'
        WHEN FLOOR(DATEDIFF(DAY, birthdate, GETDATE()) / 365.25) BETWEEN 45 AND 54 THEN '45-54'
        WHEN FLOOR(DATEDIFF(DAY, birthdate, GETDATE()) / 365.25) BETWEEN 55 AND 64 THEN '55-64'
        ELSE '65+'
    END AS age_group,
    COUNT(*) AS count
FROM  
   human_resources
WHERE 
    termdate IS NULL
GROUP BY 
    CASE
        WHEN FLOOR(DATEDIFF(DAY, birthdate, GETDATE()) / 365.25) BETWEEN 18 AND 24 THEN '18-24'
        WHEN FLOOR(DATEDIFF(DAY, birthdate, GETDATE()) / 365.25) BETWEEN 25 AND 34 THEN '25-34'
        WHEN FLOOR(DATEDIFF(DAY, birthdate, GETDATE()) / 365.25) BETWEEN 35 AND 44 THEN '35-44'
        WHEN FLOOR(DATEDIFF(DAY, birthdate, GETDATE()) / 365.25) BETWEEN 45 AND 54 THEN '45-54'
        WHEN FLOOR(DATEDIFF(DAY, birthdate, GETDATE()) / 365.25) BETWEEN 55 AND 64 THEN '55-64'
        ELSE '65+'
    END
ORDER BY 
    CASE
        WHEN FLOOR(DATEDIFF(DAY, birthdate, GETDATE()) / 365.25) BETWEEN 18 AND 24 THEN '18-24'
        WHEN FLOOR(DATEDIFF(DAY, birthdate, GETDATE()) / 365.25) BETWEEN 25 AND 34 THEN '25-34'
        WHEN FLOOR(DATEDIFF(DAY, birthdate, GETDATE()) / 365.25) BETWEEN 35 AND 44 THEN '35-44'
        WHEN FLOOR(DATEDIFF(DAY, birthdate, GETDATE()) / 365.25) BETWEEN 45 AND 54 THEN '45-54'
        WHEN FLOOR(DATEDIFF(DAY, birthdate, GETDATE()) / 365.25) BETWEEN 55 AND 64 THEN '55-64'
        ELSE '65+'
    END;


	-- -- 4. How many employees work at Headquaters vs remote
	SELECT location,
	COUNT(*) AS count
FROM human_resources
GROUP BY location;

-- 5. What is the average length of employement who have been teminated

SELECT 
    AVG(DATEDIFF(DAY, hire_date, termdate)) AS avg_length_of_employment
FROM 
    human_resources
WHERE 
    termdate IS NOT NULL AND termdate <= GETDATE();

	-- OR--

	SELECT ROUND(AVG(year(termdate) - year(hire_date)),0) AS length_of_emp
FROM human_resources
WHERE termdate IS NOT NULL AND termdate <= GETDATE()




-- 6. How does the gender distribution vary acorss dept. and job titles


SELECT
    department,
    jobtitle,
    gender,
    COUNT(*) AS count
FROM
    human_resources
WHERE
    termdate IS NULL  -- Exclude terminated employees, adjust as needed
GROUP BY
    department,
    jobtitle,
    gender
ORDER BY
    department,
    jobtitle,
    gender;


-- 7.	What is the distribution of jobtitles acorss the company

SELECT
    jobtitle,
    COUNT(*) AS count
FROM
    human_resources
WHERE
    termdate IS NULL  -- Exclude terminated employees, adjust as needed
GROUP BY
    jobtitle
ORDER BY
    jobtitle;

	-- 8. Which dept has the higher turnover/termination rate

	SELECT
    department,
    COUNT(*) AS total_employees,
    SUM(CASE WHEN termdate IS NOT NULL THEN 1 ELSE 0 END) AS terminated_employees,
    CAST(SUM(CASE WHEN termdate IS NOT NULL THEN 1 ELSE 0 END) AS FLOAT) / COUNT(*)*100,2)) AS termination_rate
FROM
    human_resources
GROUP BY
    department
ORDER BY
    termination_rate DESC;

	--9) What is the distribution of employees across location_state

	SELECT
    location_state,
    COUNT(*) AS count
FROM
    human_resources
WHERE
    termdate IS NULL  -- Exclude terminated employees, adjust as needed
GROUP BY
    location_state
ORDER BY
    count DESC;

	-- 10. How has the companys employee count changed over time based on hire and termination date.

	WITH EmployeeCounts AS (
    SELECT
        hire_date AS event_date,
        'Hire' AS event_type,
        COUNT(*) AS count
    FROM
        human_resources
    GROUP BY
        hire_date

    UNION

    SELECT
        termdate AS event_date,
        'Termination' AS event_type,
        COUNT(*) AS count
    FROM
        human_resources
    WHERE
        termdate IS NOT NULL
    GROUP BY
        termdate
)

SELECT
    event_date,
    event_type,
    SUM(count) OVER (ORDER BY event_date) AS running_total
FROM
    EmployeeCounts
ORDER BY
    event_date;


	-- 11. What is the tenure distribution for each dept.

	SELECT
    department,
    FLOOR(DATEDIFF(DAY, hire_date, COALESCE(termdate, GETDATE())) / 365.25) AS tenure_years,
    COUNT(*) AS count
FROM
    human_resources
WHERE
    termdate IS NOT NULL  -- Exclude current employees, adjust as needed
GROUP BY
    department,
    FLOOR(DATEDIFF(DAY,hire_date, COALESCE(termdate, GETDATE())) / 365.25)
ORDER BY
    department,
    tenure_years;

	