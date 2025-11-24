# Projeto de análise do catálogo da Netflix, com foco em consultas eficientes e visualização de dados em um banco de grande porte.

# Project for analyzing the Netflix catalog, focusing on efficient queries and data visualization in a large-scale database.

![](https://github.com/najirh/netflix_sql_project/blob/main/logo.png)

## Overview
This project involves a comprehensive analysis of Netflix's movies and TV shows data using SQL. The goal is to extract valuable insights and answer various business questions based on the dataset. The following README provides a detailed account of the project's objectives, business problems, solutions, findings, and conclusions.

## Objectives

- Analyze the distribution of content types (movies vs TV shows).
- Identify the most common ratings for movies and TV shows.
- List and analyze content based on release years, countries, and durations.
- Explore and categorize content based on specific criteria and keywords.

## Dataset

The data for this project is sourced from the Kaggle dataset:

## Schema

```sql
DROP TABLE IF EXISTS netflix;
CREATE TABLE netflix
(
    show_id      VARCHAR(5),
    type         VARCHAR(10),
    title        VARCHAR(250),
    director     VARCHAR(550),
    casts        VARCHAR(1050),
    country      VARCHAR(550),
    date_added   VARCHAR(55),
    release_year INT,
    rating       VARCHAR(15),
    duration     VARCHAR(15),
    listed_in    VARCHAR(250),
    description  VARCHAR(550)
);
```

## Business Problems and Solutions

### 1. Count the Number of Movies vs TV Shows

```sql
SELECT 
    type,
    COUNT(*)
FROM netflix
GROUP BY 1;
```

**Objective:** Determine the distribution of content types on Netflix.

### 2. Find the Most Common Rating for Movies and TV Shows

```sql
WITH RatingCounts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix
    GROUP BY type, rating
),
RankedRatings AS (
    SELECT 
        type,
        rating,
        rating_count,
        RANK() OVER (PARTITION BY type ORDER BY rating_count DESC) AS rank
    FROM RatingCounts
)
SELECT 
    type,
    rating AS most_frequent_rating
FROM RankedRatings
WHERE rank = 1;
```

**Objective:** Identify the most frequently occurring rating for each type of content.

### 3. List All Movies Released in a Specific Year (e.g., 2017)

```sql
SELECT * 
FROM netflix
WHERE release_year = 2017;
```

**Objective:** Retrieve all movies released in a specific year.

### 4. Find the Top 5 Countries with the Most Content on Netflix

```sql
SELECT * 
FROM
(
    SELECT 
        UNNEST(STRING_TO_ARRAY(country, ',')) AS country,
        COUNT(*) AS total_content
    FROM netflix
    GROUP BY 1
) AS t1
WHERE country IS NOT NULL
ORDER BY total_content DESC
LIMIT 5;
```

**Objective:** Identify the top 5 countries with the highest number of content items.

### 5. Identify the Longest Movie

```sql
SELECT 
    *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
```

**Objective:** Find the movie with the longest duration.

### 6. Find Content Added in the Last 5 Years

```sql
SELECT *
FROM netflix
WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years';
```

**Objective:** Retrieve content added to Netflix in the last 5 years.

### 7. Find All Movies/TV Shows by Director 'Christopher Nolan'

```sql
SELECT *
FROM
(

SELECT 
	*,
	UNNEST(STRING_TO_ARRAY(director, ',')) as director_name
FROM 
netflix
)
WHERE 
	director_name = 'Christopher Nolan'

```

**Objective:** List all content directed by 'Christopher Nolan'.

### 8. List All TV Shows with More Than 5 Seasons

```sql
SELECT *
FROM netflix
WHERE type = 'TV Show'
  AND SPLIT_PART(duration, ' ', 1)::INT > 5;
```

**Objective:** Identify TV shows with more than 5 seasons.

### 9. Count the Number of Content Items in Each Genre

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(listed_in, ',')) AS genre,
    COUNT(*) AS total_content
FROM netflix
GROUP BY 1;
```

**Objective:** Count the number of content items in each genre.

### 10.Find each year and the average numbers of content release in Brazil on netflix. 
return top 5 year with highest avg content release!

```sql
SELECT 
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        COUNT(show_id)::numeric /
        (SELECT COUNT(show_id) FROM netflix WHERE country = 'Brazil')::numeric * 100, 2
    ) AS avg_release
FROM netflix
WHERE country = 'Brazil'
GROUP BY country, release_year
ORDER BY avg_release DESC
LIMIT 5;
```

**Objective:** Calculate and rank years by the average number of content releases by Brazil.

### 11. List All Movies that are Documentaries

```sql
SELECT * 
FROM netflix
WHERE listed_in LIKE '%Documentaries';
```

**Objective:** Retrieve all movies classified as documentaries.

### 12. Find All Content Without a Director

```sql
SELECT * 
FROM netflix
WHERE director IS NULL;
```

**Objective:** List content that does not have a director.

### 13. Find how many movies actor 'Robert Downey Jr.' appeared in last 10 years!

```sql
SELECT * FROM netflix
WHERE 
	casts LIKE '%Robert Downey Jr.%'
	AND 
	release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10
```

**Objective:** Count the number of movies featuring 'Robert Downey Jr.' in the last 10 years.

### 14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in Brazil

```sql
SELECT 
    UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
    COUNT(*)
FROM netflix
WHERE country = 'Brazil'
GROUP BY actor
ORDER BY COUNT(*) DESC
LIMIT 10;
```

**Objective:** Identify the top 10 actors with the most appearances in Brazil-produced movies.

### 15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords

```sql
SELECT 
    category,
    COUNT(*) AS content_count
FROM (
    SELECT 
        CASE 
            WHEN description ILIKE '%kill%' OR description ILIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix
) AS categorized_content
GROUP BY category;
```

**Objective:** Categorize content as 'Bad' if it contains 'kill' or 'violence' and 'Good' otherwise. Count the number of items in each category.

### 16. Find the 10 titles with the longest descriptions.

```sql
SELECT 
    title,
    LENGTH(description) AS description_length
FROM netflix
ORDER BY description_length DESC
LIMIT 10;
```
**Objective:** Identify the titles with the longest description text in the catalog.
This helps highlight content where Netflix provides more detailed summaries, useful for text analysis or NLP tasks.


### 17. Find the countries that produced the most films (Movies only)

```sql
SELECT
    UNNEST(STRING_TO_ARRAY(country, ',')) AS country_name,
    COUNT(*) AS movie_count
FROM netflix
WHERE type = 'Movie'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 10;
```

**Objective:** Determine which countries have the highest film production represented in the Netflix catalog.
This analysis reveals geographic distribution and dominance in movie creation.

### 18. Identify directors with the most works in the catalog.

```sql
SELECT
    UNNEST(STRING_TO_ARRAY(director, ',')) AS director_name,
    COUNT(*) AS total_content
FROM netflix
GROUP BY 1
ORDER BY 2 DESC;

```
**Objective:** Find the directors who have the highest number of titles in the Netflix catalog, regardless of whether they are movies or series.
This allows us to identify the most prolific or frequently featured directors on the platform.


### 19. Find the 5 years with the most movie releases.

```sql

SELECT 
    release_year,
    COUNT(*) AS total_movies
FROM netflix
WHERE type = 'Movie'
GROUP BY 1
ORDER BY 2 DESC
LIMIT 5;

```

**Objective:** Identify the release years that had the highest number of movie launches.
This helps uncover trends and peak periods of film production.


### Question 20:
In this analysis, the goal is to identify directors who have worked on both films (Movies) 
and series (TV Shows) available in the Netflix catalog.

In addition to finding them, the search also organizes all titles associated 
with each director, making the result more readable and useful for analysis..

```sql
WITH Expanded AS (
    SELECT 
        show_id,
        title,
        type,
        TRIM(UNNEST(STRING_TO_ARRAY(director, ','))) AS director_name
    FROM netflix
),
DirectorsBoth AS (
    SELECT 
        director_name
    FROM Expanded
    GROUP BY director_name
    HAVING COUNT(DISTINCT type) = 2
)
SELECT 
    e.director_name,
    ARRAY_AGG(e.title ORDER BY e.title) FILTER (WHERE e.type = 'Movie') AS movies,
    ARRAY_AGG(e.title ORDER BY e.title) FILTER (WHERE e.type = 'TV Show') AS tv_shows
FROM Expanded e
JOIN DirectorsBoth d ON e.director_name = d.director_name
GROUP BY e.director_name
ORDER BY e.director_name;
```
**Objective:** Find directors who have worked on both movies and TV shows within the Netflix catalog.
The query also organizes all titles associated with each qualifying director, clearly separating movies and TV shows to improve readability and analytical usefulness.



## Findings and Conclusion

- **Content Distribution:** The dataset contains a diverse range of movies and TV shows with varying ratings and genres.
- **Common Ratings:** Insights into the most common ratings provide an understanding of the content's target audience.
- **Geographical Insights:** The top countries and the average content releases by India highlight regional content distribution.
- **Content Categorization:** Categorizing content based on specific keywords helps in understanding the nature of content available on Netflix.

This analysis provides a comprehensive view of Netflix's content and can help inform content strategy and decision-making.



## Author - Isaac Mendes dos Santos

This project is part of my portfolio and reflects the SQL skills I’ve been developing for data analysis. If you want to ask something, leave feedback, or work together, just reach out!

### Stay Updated and Join the Community

For more content on SQL, data analysis, and other data-related topics, make sure to follow me on social media and join our community:

- **Instagram**: [Follow me for daily tips and updates](https://www.instagram.com/santosmds_isaac/)
- **LinkedIn**: [Connect with me professionally](https://www.linkedin.com/in/isaac-mendes-dos-santos-20258b249/)

Thank you for your support, and I look forward to connecting with you!
