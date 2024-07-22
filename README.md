
# Introduction

This capstone project is part of my journey to becoming a data analyst, developed during my SQL course. 
The main goal is to show my skills in writing SQL queries to pull and analyze data from databases. 
In this project, I focus on finding the top-paying jobs for Data Analysts, figuring out which skills are in high demand,
calculating the average salaries for these skills, and discovering where high demand meets high salary. 
I hope to provide valuable insights into the Data Analyst job market, highlighting the skills that offer the best financial rewards.


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

- 1-  Identify the top 10 highest-paying Data Analyst roles that are available remotely. 

To identify the top 10 highest-paying jobs from the available tables, I used a LEFT JOIN to return all records from the left table (job_postings_fact)
 and the matching rows from the right table (company_dim) based on the common column job_id. I filtered the results to include only 
 remote positions for Data Analysts and excluded any rows with NULL values. Additionally, I included the company names offering these positions in the SELECT clause.


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

In this query, I used a CTE (Common Table Expression) to reference the results from Query 1,
 creating a temporary table called 'top_paying_jobs'. In the main query, I then extracted the skills 
 required for the top 10 highest-paying jobs.

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

To identify the jobs that are in high demand, I counted the frequency of each 
skill listed in the skills column. Then, I grouped the results by skill to consolidate the data.


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

The main goal of Query 4 is to associate skills with high salaries. To achieve this,
I performed an INNER JOIN between the skills_job_dim and job_posting_facts tables using the 
'job_id' column to match skills with the salaries for each position. Then, I calculated the 
average salaries for Data Analyst roles, excluding NULL values.
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

In Query 5, the strategy is to combine the results of Query 3 (which identifies the most in-demand skills) and Query 4
(which associates skills with higher salaries). I could have used both with CTE function, 
but I opt for a shorter query, which gave me the results I was looking for. 

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

# What I learned

Through my journey with the SQL for Data Analytics certificate by Luke Barousse, I've learned everything from basic operations to advanced features in SQL, including:

- **Features**: Comparison, operations, aggregation, and JOIN.
- **Data Cleaning**: How to exclude NULL values.
- **Data Manipulation**: How to create databases.
- **Database Loading**: Loading a database using PostgreSQL and VSCode.
- **Complex Query Construction**: Building advanced SQL queries that combine multiple tables.


# Conclusion
### Insights
- A Data Analyst can earn up to 650,000 per year.
- High-demand skills include SQL, Excel, Python, Tableau, and Power BI.
- Skills associated with higher salaries are PySpark, Bitbucket, Watson, Couchbase, and DataRobot.
- Skills that combine high demand with higher salaries include SQL, Excel, Python, Tableau, and R.


### Closing Toughts

This project significantly enhanced my SQL abilities and provided valuable insights into the Data Analytics 
job market. It not only taught me how to code and construct queries but also developed my critical and analytical 
thinking skills. I learned how to approach problems effectively and leverage data to drive better results. 
As this project fueled my growing interest in data analysis, I am becoming more confident in my SQL skills and 
motivated to deepen my knowledge further on my path to becoming a Data Analyst.

