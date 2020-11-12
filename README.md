# SQL Analysis With BIGQUERY and Dashboard in Data Studio
This project helped me to apply the basics of SQL and get acquainted with bigger public data sets in BIGQUERY and Data Studio. 

Data was analyzed on 09/17/2020.

Data set: https://console.cloud.google.com/bigquery?_ga=2.235996418.22298598.1603233578-1826038417.1594837092&_gac=1.87764330.1600309907.CjwKCAjw74b7BRA_EiwAF8yHFM72pOdybSuzR6CPhsAwKKsN5xsrW3QvEb7DwXqHDvlSAX_zbcRtnBoCTi8QAvD_BwE&project=crested-archive-225711&folder=&organizationId=&p=bigquery-public-data&d=stackoverflow&page=dataset

## **Business Questions**

### 1. How many users are there in total?

```sql
SELECT 
  COUNT(id)
FROM 
  `bigquery-public-data.stackoverflow.users`
```

**Query explanation**

By counting the total ID numbers, we will get to know how many users there are in total.

### 2. How many users per year?

```sql
-- How many new users are there? 
SELECT
  EXTRACT(YEAR
  FROM
    creation_date) AS year,
  COUNT(*) AS new_users,
FROM
  `bigquery-public-data.stackoverflow.users`
GROUP BY
  year
ORDER BY
  year;

-- How many active users were there in this year
SELECT
  EXTRACT(YEAR
  FROM
    last_access_date) AS last_access_year,
  COUNT(*) AS users,
FROM
  `bigquery-public-data.stackoverflow.users`
GROUP BY
  last_access_year
ORDER BY
 last_access_year;
```

**Query explanation**

We get the year with the extract function to get a year value only. This is needed as we work with a timestamp data type. This is saved as the column name year (or last access date). We want to count the new or last used users by selecting all the rows and group them by the year. Then we order it by the year so that there is a sense of time present in the data. This helps me to create a line chart in data studio.

### 3. How many questions per day/month/year? Use filter

```sql
SELECT
  CAST(creation_date AS date) AS date, 
  COUNT(title) AS questions_per_day
FROM
  `bigquery-public-data.stackoverflow.posts_questions`
GROUP BY
  date
ORDER BY
  date ASC;
```

**Query explanation**

When using cast, we can get a date data type rather than the timestamp when we count the title, which will give us the number of questions. Because we group it by date and order it by date, the number of questions per day will be shown by date (day/month/year). This could be filtered within the data studio.

### 4. Most popular tags by month

**4.1. Optional Solution (input by month & year)**

```sql
SELECT 
  tags, 
  COUNT(title) AS number_of_tags_used, 
   EXTRACT(MONTH 
   FROM creation_date) AS month, 
   EXTRACT(YEAR 
   FROM creation_date) AS year
FROM
  `bigquery-public-data.stackoverflow.posts_questions`
WHERE  
   EXTRACT(MONTH 
   FROM 
       creation_date) = 9 
   AND EXTRACT(YEAR 
   FROM creation_date) = 2008
GROUP BY 
   tags, 
   month, 
   year
ORDER BY 
   number_of_tags_used DESC
LIMIT 
   3;
```

**Query explanation**

In this example, the tags will be presented together with the title of the questions. This gives us a result of the number of questions per tag, assuming that the popularity of a tag is defined by the number of questions. I extracted the year and month because assumed that once you want to see the month, the user would also like to know in which year that is in. With this query, we will only get the result for a specific year and month. That is why I used the where condition clause. I grouped it by the tags, then the month, and year. Ordered it by the number of tags used to see which one is the Highest. By limiting it, you get the top 3 results.

**4.2 Actual Solution**

```sql
SELECT
    ARRAY_AGG(t
    ORDER BY
      number_of_times_used DESC
    LIMIT
      3) top_tags
  FROM (
    SELECT
      EXTRACT(YEAR
      FROM
        creation_date) AS year,
      EXTRACT(MONTH
      FROM
        creation_date) AS month,
      tags,
      COUNT(title) AS number_of_times_used
    FROM
      `bigquery-public-data.stackoverflow.posts_questions`
    GROUP BY
      year,
      month,
      tags ) t
  GROUP BY
    year,
    month ) t,
  t.top_tags
ORDER BY
  year DESC,
  month DESC,
  number_of_times_used DESC
```

**Query explanation**

I am not going to lie; I did get some help with this one via Stack Overflow. A top_hits metric aggregator keeps track of the most relevant document being aggregated. This is this case if the top three tags for each category. In order to create the condition, I created (with help) a subquery containing the year, month, the tags, and the count of the titles of questions (as a factor of popularity). Within the subquery, I grouped it by tags with t, the year and the month. In in the overall query, these he same for the overall group by function. However, when ordering by, we use the number of times again.

### 5. What comments are the most voted this week?

```sql
SELECT
  pq.title,
  c.text AS Comment, 
  c.score AS Total_vote_score
FROM
  `bigquery-public-data.stackoverflow.comments` AS c
INNER JOIN `bigquery-public-data.stackoverflow.posts_questions` AS pq
ON c.post_id = pq.id
WHERE
  c.creation_date <'2020-05-31'
  AND c.creation_date >'2020-05-24'
  AND c.text IS NOT NULL
ORDER BY
  c.score DESC
LIMIT 
   20
```

**Query explanation**

By joining the tables, I could get a more comprehensive overview of the comments. I selected the title of the post_question table, which gives me the actual question that has been asked. Then I decided on the text from the comment table, which provides me with the comment that has been posted. I also selected the score of this table, which is the overall voting score (sum of + and – votes). I only wanted to see the rows that are matching with each other; therefore, I used an inner join connecting them on the post_id within the comments table and id within the posts_questions table. Because we want to know the votes in a specific week, I selected the dates from the comment table (when the comments were posted) with a where clause. I also wanted to make sure there were not empty comments in the query. By setting a limit, only the highest vote comments would be shown.
