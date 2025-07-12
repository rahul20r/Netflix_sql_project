# Netflix Movies and TV shows Data Analysis using SQL
![Netflix Logo](https://github.com/rahul20r/Netflix_sql_project/blob/861c8ce6d8dcbd699a58b613cf9969004a999375/Logo.jpg)

## Overview

This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.


## Business Problems and Solutions

SELECT * 
FROM netflix.netflix_titles

-- 15 Business Problems & Solutions

## 1. Count the number of movies vs TV Shows

select 
	type,
	count(*) as total_content
from netflix_titles
group by type
    
## 2. Find the most common rating for movies and TV shows

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
    
    
## 3. List all movies released in a specific year (e.g., 2020)

-- filter 2020
-- movies

SELECT *
 FROM netflix_titles
 WHERE type = 'movie'
 AND
 release_year = 2020


## 4. Find the top 5 countries with the most content on Netflix


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

## 5. Identify the longest movie or TV show duration

SELECT *
FROM netflix_titles
WHERE type = 'Movie'
ORDER BY CAST(SUBSTRING_INDEX(duration, ' ', 1) AS UNSIGNED) DESC
LIMIT 10;


## 6. Find content added in the last 5 years

SELECT date_added
FROM netflix_titles
WHERE STR_TO_DATE(date_added, '%M %d, %Y') >= CURDATE() - INTERVAL 5 YEAR;

## 7. Find all the movies/TV shows by director 'Toshiyuki Tsuru'!

SELECT *
FROM netflix_titles
WHERE type = 'Movie'
WHERE director like  '%Toshiyuki Tsuru%'

## 8. List all TV shows with more than 5 seasons

SELECT *
FROM netflix_titles
WHERE 
    type = 'TV Show'
    AND CAST(SUBSTRING_INDEX(duration, ' ', 1) AS UNSIGNED) > 5;


## 9. Count the number of content items in each genre



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


## 10. Find the average release year for content produced in a specific country
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


## 11. List all movies that are documentaries

SELECT *
FROM netflix_titles
WHERE listed_in LIKE '%Documentaries%';


## 12. Find all content without a director

SELECT *
FROM netflix_titles
ORDER BY Director


## 13. Find how many movies actor 'Junko Takeuchi'

SELECT *
FROM netflix_titles
WHERE cast LIKE '%Junko Takeuchi%'

## 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.

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


## 15.Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category.

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


**Objective:** Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.

## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.
