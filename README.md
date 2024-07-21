
# Introduction

This project was build as a capstone project for my SQL course on my hourney to become a data analyst in order to demonstrate my skills in writing queries to abtain information of databases. 
The main goal of this project is to annalyze the top paying jobs for Data Analyst, skills in high demand and their average salary, and where high demands meets high salary. 
SQL Queries? Check them out here : [project_sql folder](/sql_load/)

# Background
The motivation behind this project stemmed from my desire to understand the data analyst job market better. I aimed to discover which skills are paid the most and in demand, making my job search more targeted and effective. 

The data for this analysis is from Luke Barousse’s SQL Course (include link to the course). This data includes details on job titles, salaries, locations, and required skills. 

The questions I wanted to answer through my SQL queries were:

1. What are the top-paying data analyst jobs?
2. What skills are required for these top-paying jobs?
3. What skills are most in demand for data analysts?
4. Which skills are associated with higher salaries?
5. What are the most optimal skills to learn for a data analyst looking to maximize job market value?


# Tools I Used

In this project, I utilized a variety of tools to conduct my analysis:

- **SQL** (Structured Query Language): Enabled me to interact with the database, extract insights, and answer my key questions through queries.
- **PostgreSQL**: As the database management system, PostgreSQL allowed me to store, query, and manipulate the job posting data.
- **Visual Studio Code:** This open-source administration and development platform helped me manage the database and execute SQL queries.

# Analysis

Each query for this project aimed at investigating specific aspects of the data analyst job market. Here’s how I approached each question:

- 1-  Identify the top 10 highest-paying Data Analyst roles that are available remotely. 

To identify the top 10 paying jobs in the tables availables, I used Left Join to returns all records from the left table (job_postings_fact), and the matching rows from the right table (company_dim) through the common row job_id.
I specified that I wanted only remote positions for data analyst role, and also to not show me NULL results
I also wanted to know what companies are offering these positions so I added on my SELECT section the name of them

```sql
-- Top 10 paying Data Analyst role
SELECT
    job_postings_fact.job_id,
    job_postings_fact.company_id,
    job_postings_fact.job_title,
    job_postings_fact.job_location,
    job_postings_fact.salary_year_avg,
    company_dim.name
FROM 
    job_postings_fact
LEFT JOIN 
    company_dim ON job_postings_fact.company_id = company_dim.company_id
WHERE 
    job_work_from_home = TRUE 
    AND job_title_short = 'Data Analyst'
    AND salary_year_avg IS NOT NULL
ORDER BY 
    salary_year_avg DESC

LIMIT 10
```
![alt text](<sql_load/Screenshot 2024-07-21 at 2.50.34 PM.jpg>)

- 2- What are the top-paying data analyst jobs, and what skills are required?

In this query I used a CTE to reference results from Query 1, creating a temporary table called 'top_paying_jobs' .
Then in the main query I find the skills required for the top 10 paying jobs. 
```SQL
-- Getting the top 10 paying Data Analyst jobs
WITH top_paying_jobs AS(
    SELECT
    job_postings_fact.job_id,
    job_postings_fact.job_title,
    job_postings_fact.job_location,
    job_postings_fact.salary_year_avg,
    company_dim.name
FROM 
    job_postings_fact
LEFT JOIN 
    company_dim ON job_postings_fact.company_id = company_dim.company_id

WHERE job_work_from_home = TRUE 
    AND job_title_short = 'Data Analyst'
    AND salary_year_avg IS NOT NULL

ORDER BY 
    salary_year_avg DESC

LIMIT 10
)
-- Skills Required for Data Analyst position
SELECT 
    top_paying_jobs.job_id,
    top_paying_jobs.job_title,
    top_paying_jobs.job_location,
    top_paying_jobs.salary_year_avg,
    skills_dim.skill_id, 
    skills_dim.skills

FROM 
    top_paying_jobs

    INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id

ORDER BY top_paying_jobs.salary_year_avg DESC
```
Note: this tabbles are jus the the 5 first jobs
![alt text](<Asset/Screenshot 2024-07-21 at 4.59.33 PM.png>)
![alt text](<Asset/Screenshot 2024-07-21 at 5.00.15 PM.png>)



- 3- What skills are most in demand for data analysts?

To get what jobs are in high demand, I had to count how many times the skills appears on the skills row. 


``` SQL
-- Skills in demand for Data Analyst role
SELECT
    skills_dim.skills,
    COUNT(skills_dim.skills) AS skills_used_most
FROM job_postings_fact
    INNER JOIN 
        skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN
        skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id
WHERE job_postings_fact.job_title_short = 'Data Analyst'

GROUP BY skills_dim.skills

ORDER BY skills_used_most DESC

LIMIT 10
```
![alt text](<Asset/Screenshot 2024-07-21 at 3.40.57 PM.png>)

- 4-Which skills are associated with higher salaries?

Query number 4 main goal is to associate skills with high salaries.
So I INNER JOIN skills_job_dim on job_posting_facts through 'job_id' row,to match skills with the salaries for each position
Then took the avg of the salaries for data analyst, excluing NULL values.
``` SQL
-- Skills associated with higher salaries
SELECT
    skills_dim.skills,
    AVG(job_postings_fact.salary_year_avg) AS salary_year_avg

FROM job_postings_fact

    INNER JOIN 
        skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN
        skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id

WHERE 
    job_postings_fact.job_title_short = 'Data Analyst'
    AND job_postings_fact.salary_year_avg IS NOT NULL
    AND job_postings_fact.job_work_from_home = TRUE

GROUP BY skills_dim.skills

ORDER BY salary_year_avg DESC

LIMIT 10
```
![alt text](<Asset/Screenshot 2024-07-21 at 3.31.32 PM.png>)

- 5-What are the most optimal skills to learn (aka it’s in high demand and a high-paying skill) for a data analyst?

Query number 5, the strategy is to cross results of query number 3,What skills are most in demand, and query 4, Which skills are associated with higher salaries. 
I started by selecting rows I needed, and counting how many jobs there were available,
``` SQL
SELECT
    skills_dim.skill_id,
    skills_dim.skills,
    COUNT(skills_job_dim.job_id) AS demand_count,
    AVG(job_postings_fact.salary_year_avg) AS salary_year_avg
     

FROM job_postings_fact
    INNER JOIN 
        skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN
        skills_dim ON skills_job_dim.skill_id = skills_dim.skill_id

WHERE 
    job_postings_fact.job_title_short = 'Data Analyst'
    AND job_postings_fact.salary_year_avg IS NOT NULL
    AND job_postings_fact.job_work_from_home = TRUE

GROUP BY skills_dim.skill_id

HAVING 
    COUNT(skills_job_dim.job_id) >10


ORDER BY salary_year_avg DESC
```
![alt text](<Asset/5.1.png>)
![alt text](<Asset/5.2.png>)