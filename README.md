# Netflix Movies and TV shows Data Analysis using SQL
![Netflix Logo](https://github.com/rahul20r/Netflix_sql_project/blob/861c8ce6d8dcbd699a58b613cf9969004a999375/Logo.jpg)


## Business Problems

-- 15 Business Problems & Solutions
# Netflix Movies and TV Shows Data Analysis using SQL

## Overview

This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.


## Business Problems and Solutions

### 1. Count the Number of Movies vs TV Shows

```sql
select 
	type,
	count(*) as total_content
from netflix_titles
group by type
```

**Objective:** Determine the distribution of content types on Netflix.

### 2. Find the Most Common Rating for Movies and TV Shows

```sql
SELECT
	type,
    rating
FROM

(select
	type,
    rating,
    count(*),
    RANK() OVER (PARTITION BY type ORDER BY COUNT(*) DESC) AS ranking
    from netflix_titles
group by 1,2
) AS t1
WHERE
	ranking = 1

```

**Objective:** Identify the most frequently occurring rating for each type of content.

### 3. List All Movies Released in a Specific Year (e.g., 2020)

```sql
-- filter 2020
-- movies

SELECT *
 FROM netflix_titles
 WHERE type = 'movie'
 AND
 release_year = 2020

```

**Objective:** Retrieve all movies released in a specific year.

### 4. Find the Top 5 Countries with the Most Content on Netflix

```sql

SELECT 
	country,
    COUNT(show_id) as total_content
FROM netflix.netflix_titles
group by 1


WITH RECURSIVE split_country AS (
    SELECT 
        show_id,
        TRIM(SUBSTRING_INDEX(country, ',', 1)) AS country,
        SUBSTRING(country, LENGTH(SUBSTRING_INDEX(country, ',', 1)) + 2) AS remaining
    FROM netflix.netflix_titles
    WHERE country IS NOT NULL

    UNION ALL

    SELECT 
        show_id,
        TRIM(SUBSTRING_INDEX(remaining, ',', 1)),
        SUBSTRING(remaining, LENGTH(SUBSTRING_INDEX(remaining, ',', 1)) + 2)
    FROM split_country
    WHERE remaining IS NOT NULL AND remaining != ''
)

SELECT 
    country,
    COUNT(DISTINCT show_id) AS total_content
FROM split_country
GROUP BY 1
ORDER BY 2 DESC;

```

**Objective:** Identify the top 5 countries with the highest number of content items.

### 5. Identify the Longest Movie or TV Show Duration

```sql
SELECT *
FROM netflix_titles
WHERE type = 'Movie'
ORDER BY CAST(SUBSTRING_INDEX(duration, ' ', 1) AS UNSIGNED) DESC
LIMIT 10;

```

**Objective:** Find the movie with the longest duration.

### 6. Find Content Added in the Last 5 Years

```sql
SELECT date_added
FROM netflix_titles
WHERE STR_TO_DATE(date_added, '%M %d, %Y') >= CURDATE() - INTERVAL 5 YEAR;
```

**Objective:** Retrieve content added to Netflix in the last 5 years.

### 7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

```sql
SELECT *
FROM netflix_titles
WHERE type = 'Movie'
WHERE director like  '%Toshiyuki Tsuru%'
```

**Objective:** List all content directed by 'Rajiv Chilaka'.

### 8. List All TV Shows with More Than 5 Seasons

```sql
SELECT *
FROM netflix_titles
WHERE 
    type = 'TV Show'
    AND CAST(SUBSTRING_INDEX(duration, ' ', 1) AS UNSIGNED) > 5;
```

**Objective:** Identify TV shows with more than 5 seasons.

### 9. Count the Number of Content Items in Each Genre

```sql


WITH split_genres AS (
    SELECT show_id, TRIM(SUBSTRING_INDEX(listed_in, ',', 1)) AS genre FROM netflix_titles
    UNION ALL
    SELECT show_id, TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(listed_in, ',', 2), ',', -1)) FROM netflix_titles
    UNION ALL
    SELECT show_id, TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(listed_in, ',', 3), ',', -1)) FROM netflix_titles
    UNION ALL
    SELECT show_id, TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(listed_in, ',', 4), ',', -1)) FROM netflix_titles
    UNION ALL
    SELECT show_id, TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(listed_in, ',', 5), ',', -1)) FROM netflix_titles
)
SELECT 
    genre,
    COUNT(show_id) AS total_content
FROM split_genres
WHERE genre <> ''
GROUP BY genre
ORDER BY total_content DESC;

```

**Objective:** Count the number of content items in each genre.

### 10. Find Each Year and the Average Number of Content Releases by India on Netflix

```sql
SELECT
  YEAR(STR_TO_DATE(date_added, '%M %d, %Y')) AS year,
  COUNT(*) AS yearly_content,
  ROUND(
    (COUNT(*) / (SELECT COUNT(*) FROM netflix_titles WHERE country = 'India')) * 100,
    2
  ) AS avg_content_per_year
FROM netflix_titles
WHERE country = 'India' AND date_added IS NOT NULL
GROUP BY year
ORDER BY avg_content_per_year DESC
LIMIT 5;

```

**Objective:** Calculate and rank years by the average number of content releases by India.

### 11. List All Movies that are Documentaries

```sql
SELECT *
FROM netflix_titles
WHERE listed_in LIKE '%Documentaries%';

```

**Objective:** Retrieve all movies classified as documentaries.

### 12. Find All Content Without a Director

```sql
SELECT *
FROM netflix_titles
ORDER BY Director
```

**Objective:** List content that does not have a director.

### 13. Find How Many Movies Actor 'Junko Takeuchi' Appeared in the Last 10 Years

```sql

SELECT *
FROM netflix_titles
WHERE cast LIKE '%Junko Takeuchi%'
```

**Objective:** Count the number of movies featuring 'Junko Takeuchi'' in the last 10 years.

### 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

```sql
WITH RECURSIVE numbers AS (
  SELECT * 1 AS n
  UNION ALL
  SELECT n + 1 FROM numbers WHERE n <= 10  -- Increase if needed
)
SELECT *
  TRIM(SUBSTRING_INDEX(SUBSTRING_INDEX(cast, ',', n), ',', -1)) AS actor,
  COUNT(*) AS total_content
FROM netflix_titles
JOIN numbers
  ON n <= 1 + LENGTH(cast) - LENGTH(REPLACE(cast, ',', ''))
WHERE country LIKE '%India%' 
  AND cast IS NOT NULL
GROUP BY actor
ORDER BY total_content DESC
LIMIT 10;
```

**Objective:** Identify the top 10 actors with the most appearances in Indian-produced movies.

### 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords

```sql

        WITH new_table AS (
  SELECT *,
    CASE
      WHEN LOWER(description) LIKE '%kill%' OR LOWER(description) LIKE '%violence%' THEN 'Bad_content'
      ELSE 'Good Content'
    END AS category
  FROM netflix_titles
)
SELECT
  category,
  COUNT(*) AS total_content
FROM new_table
GROUP BY category;
```

**Objective:** Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.

## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.

**Objective:** Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.

## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.
