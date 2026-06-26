## Introduction
The data job market has become one of the fastest-growing sectors in today's technology-driven economy. As organizations generate and rely on increasing amounts of data, professionals such as Data Analysts, Data Scientists, Data Engineers, Business Intelligence Analysts, and Machine Learning Engineers have become essential for transforming raw information into valuable insights and business solutions.

This project analyzes the current data-related job market using real-world job posting data to identify industry trends and employment opportunities. The analysis focuses on key aspects such as the most in-demand job roles, required technical skills, salary ranges, remote work availability, and the companies actively hiring data professionals.

SQL Queries here: [project_sql folder](/proyecto_sql/)

## Tools Used

This project was developed using several industry-standard tools for data analysis and database management. **SQL** was the primary language used to query, filter, aggregate, and analyze the job market data, allowing meaningful insights to be extracted from the dataset.

**PostgreSQL** served as the database management system, providing a reliable and efficient environment for storing and managing large volumes of job posting data. Its advanced SQL capabilities made it well-suited for performing complex queries and data analysis.

**Visual Studio Code (VS Code)** was used as the main development environment to write and organize SQL scripts, making the workflow more efficient through features such as syntax highlighting, extensions, and integrated version control.

 **GitHub** was used for version control and project management. It enabled the tracking of changes throughout the development process, ensured backup of the project files, and facilitated collaboration and code sharing.

## Analysis
### Top paying jobs
This analysis identifies the highest-paying data-related positions by examining the average salaries listed in job postings. It highlights which roles offer the greatest earning potential and provides insight into how compensation varies across different careers within the data industry, helping job seekers understand which positions are the most financially rewarding.

```sql
SELECT
    job_id,
    job_title,
    job_location,
    job_schedule_type,
    salary_year_avg,
    job_posted_date,
    name as company_name
FROM
    job_postings_fact
LEFT JOIN company_dim on job_postings_fact.company_id = company_dim.company_id
WHERE
    job_title_short = 'Data Analyst' AND
    job_location = 'Anywhere' AND 
    salary_year_avg IS NOT NULL
ORDER BY
    salary_year_avg DESC
LIMIT 10;
```

**Key Findings**

* **Highest Salary:** Data Analyst at **Mantys** with an annual salary of **$650,000**.
* **Leadership Pays More:** Roles such as **Director of Analytics** and **Associate Director – Data Insights** rank among the highest-paying positions.
* **Senior Expertise is Rewarded:** **Principal Data Analyst** positions consistently offer salaries above **$180,000** per year.
*  **100% Remote:** Every job in the top 10 is listed as **full-time** and **remote ("Anywhere")**, reflecting the strong demand for experienced data professionals in remote work environments.

![Top paying roles](assets\Top_10_Highest-Paying_Data_Jobs.png)

### Top paying jobs skills
This analysis examines the skills most commonly required for the highest-paying data jobs. By identifying the technologies and competencies associated with these positions, it helps determine which skills are valued by employers offering the highest salaries.
```sql
WITH top_paying_jobs as ( 
    SELECT
        job_id,
        job_title,
        salary_year_avg,
        job_posted_date,
        name as company_name
    FROM
        job_postings_fact
    LEFT JOIN company_dim on job_postings_fact.company_id = company_dim.company_id
    WHERE
        job_title_short = 'Data Analyst' AND
        job_location = 'Anywhere' AND 
        salary_year_avg IS NOT NULL
    ORDER BY
        salary_year_avg DESC
    LIMIT 10
)

SELECT 
    top_paying_jobs.*,
    skills
FROM top_paying_jobs
INNER JOIN skills_job_dim ON top_paying_jobs.job_id = skills_job_dim.job_id
INNER JOIN skills_dim on skills_job_dim.skill_id = skills_dim.skill_id
ORDER BY
    salary_year_avg DESC
```

**Key Findings**

* The highest-paying jobs consistently require **SQL** and **Python**, making them the most valuable core skills.
* Cloud platforms such as **AWS** and **Azure**, along with **Databricks** and **PySpark**, are common in senior-level positions.
* Business intelligence tools like **Tableau**, **Power BI**, and **Excel** remain important, even for high-paying leadership roles.
* The results show that top-paying jobs require a combination of programming, cloud computing, and data visualization skills.

![Top paying jobs skills](assets\Skills_Required_in_Top-Paying_Jobs.png)

### Top demanded skills
This section focuses on the skills that appear most frequently across all data-related job postings. It reveals the technologies and tools that employers consistently seek, providing a clear picture of the core competencies required to remain competitive in the data job market.
```sql
SELECT
    skills,
    COUNT(skills_job_dim.job_id) as demand_count
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim on skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short='Data Analyst'
    AND job_work_from_home = TRUE
GROUP BY
    skills
ORDER BY
    demand_count DESC
limit 5
```
**Key Findings**

* **SQL** is the most requested skill, appearing in **7,291** job postings.
* **Excel** and **Python** rank second and third, highlighting the importance of both spreadsheet analysis and programming.
* **Tableau** and **Power BI** are the leading data visualization tools in demand.
* Overall, employers prioritize a strong foundation in data querying, analysis, and visualization.

![Most demanded skills](assets\Top_Most_Demanded_Skills.png)

### Top paying skills
This analysis ranks individual skills based on the average salaries of the jobs that require them. It highlights which technical skills are associated with higher compensation, helping professionals prioritize learning opportunities that can maximize their earning potential.
```sql
SELECT
    skills,
    round(AVG(salary_year_avg), 0) as avg_salary
FROM job_postings_fact
INNER JOIN skills_job_dim ON job_postings_fact.job_id = skills_job_dim.job_id
INNER JOIN skills_dim on skills_job_dim.skill_id = skills_dim.skill_id
WHERE
    job_title_short='Data Analyst'
    AND salary_year_avg is NOT NULL
    -- AND job_work_from_home = TRUE
GROUP BY
    skills
ORDER BY
    avg_salary DESC
limit 50
```

**Key Findings**

* Specialized technologies command the highest salaries, with **SVN**, **Solidity**, and **Couchbase** leading the ranking.
* Skills related to machine learning and big data, such as **PyTorch**, **TensorFlow**, **Kafka**, and **Databricks**, are associated with high-paying roles.
* Cloud and data engineering technologies, including **Terraform**, **Spark**, **Snowflake**, and **GCP**, also offer strong earning potential.
* These results suggest that niche and advanced technical skills are often rewarded with higher salaries.

![Top paying skills](assets\Highest-Paying_Skills.png)

### Optimal skills
This section combines salary and demand analyses to identify the most valuable skills in the data job market. These are skills that are both highly sought after by employers and associated with competitive salaries, making them excellent choices for professionals looking to improve their career prospects.
```sql
WITH skills_demand AS (
    SELECT
        skills_dim.skill_id,
        skills_dim.skills,
        COUNT(skills_job_dim.job_id) AS demand_count
    FROM job_postings_fact
    INNER JOIN skills_job_dim
        ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim
        ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst'
        AND job_work_from_home = TRUE
        AND salary_year_avg IS NOT NULL
    GROUP BY
        skills_dim.skill_id,
        skills_dim.skills
),

avg_salary AS (
    SELECT
        skills_job_dim.skill_id,
        skills_dim.skills,
        ROUND(AVG(job_postings_fact.salary_year_avg), 0) AS avg_salary
    FROM job_postings_fact
    INNER JOIN skills_job_dim
        ON job_postings_fact.job_id = skills_job_dim.job_id
    INNER JOIN skills_dim
        ON skills_job_dim.skill_id = skills_dim.skill_id
    WHERE
        job_title_short = 'Data Analyst'
        AND salary_year_avg IS NOT NULL
        AND job_work_from_home = TRUE
    GROUP BY
        skills_job_dim.skill_id,
        skills_dim.skills
)

SELECT
    skills_demand.skill_id,
    skills_demand.skills,
    skills_demand.demand_count,
    avg_salary.avg_salary
FROM skills_demand
INNER JOIN avg_salary ON skills_demand.skill_id = avg_salary.skill_id
WHERE demand_count > 10
ORDER BY
    avg_salary DESC,
    demand_count DESC
LIMIT 25
```
**Key Findings**

* **Python** and **Tableau** stand out by combining high demand with competitive salaries.
* Cloud platforms such as **AWS**, **Azure**, and **Snowflake** provide an excellent balance between market demand and earning potential.
* Database technologies like **Oracle** and **SQL Server** remain valuable skills for data professionals.
* Developing skills that are both in demand and well compensated can provide the best long-term career opportunities in the data job market.

![Optimal skills](assets\Top_Optimal_Skills.png)

## Conclusions
This project provided valuable insights into the current data-related job market by analyzing real-world job posting data using SQL and PostgreSQL. Through the exploration of salary trends, job demand, and required skills, it became clear that technical expertise plays a significant role in both employability and earning potential within the data industry.

The analysis showed that while some skills are highly demanded across a wide range of positions, others are more closely associated with higher salaries. By combining salary and demand metrics, it was also possible to identify optimal skills that offer both strong career opportunities and competitive compensation. These findings can help students, job seekers, and professionals make more informed decisions about which technologies and competencies to prioritize in their learning journey.