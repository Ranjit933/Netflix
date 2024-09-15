# Netflix For Data Analysis using Python and Sql server 

Netflix Data Analysis and Recommendation System
# This project involves analyzing Netflix data and developing a recommendation system using Python. The main objectives are to explore trends in Netflix content, build a recommendation engine, and create an interactive dashboard for dynamic data exploration.

# Key Features:
Data Collection and Preparation:

Collected and cleaned data from Netflix’s public datasets.
Preprocessed data for analysis, including handling missing values and normalizing formats.
Exploratory Data Analysis (EDA):

Performed data visualization to uncover trends and patterns such as genre distribution and popularity over time.
Used libraries like Matplotlib and Seaborn for creating insightful charts and graphs.
Recommendation System:

Implemented collaborative filtering to recommend content based on user preferences.
Developed a content-based recommendation engine using movie/show attributes like genres and actors.
Predictive Modeling:

Built predictive models to estimate ratings and viewer engagement.
Evaluated model performance using metrics like RMSE and accuracy.
Interactive Dashboard:

Created an interactive dashboard using Plotly Dash or Streamlit for users to explore data insights dynamically.
Incorporated features such as search functionality, filters, and dynamic visualizations.
Technologies Used:
* Python
* Pandas
* NumPy
* Matplotlib
* Seaborn
* Plotly Dash/Streamlit

# SQL SERVER

CREATE DATABASE Netflix

USE Netflix

# -- Netflix Data Analysis using SQL
# -- Solutions of 15 business problems

SELECT * FROM netflix_titles

SELECT COUNT(*) AS Total_content
FROM netflix_titles

-- 1. Count the number of Movies vs TV Shows

SELECT type,COUNT(*) AS Total_content
FROM netflix_titles
GROUP BY type

-- 2. Find the most common rating for movies and TV shows

WITH RatingCounts AS (
    SELECT 
        type,
        rating,
        COUNT(*) AS rating_count
    FROM netflix_titles
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

-- 3. List all movies released in a specific year (e.g., 2020)
-- filter 2020
-- movies

SELECT * FROM netflix_titles
WHERE 
    type = 'Movie'
	AND
	release_year = 2020

-- 4. Find the top 5 countries with the most content on Netflix


WITH CountrySplit AS (  -- Split the comma-separated 'country' values into individual rows
    SELECT
        TRIM(value) AS country
    FROM
        netflix_titles
    CROSS APPLY
        STRING_SPLIT(country, ',')
),

CountryCount AS (  -- Count the occurrences of each country
    SELECT
        country,
        COUNT(*) AS total_content
    FROM
        CountrySplit
    WHERE
        country IS NOT NULL
    GROUP BY
        country
)

SELECT    -- Select the top 5 countries by content count
    country,
    total_content
FROM
    CountryCount
ORDER BY
    total_content DESC
OFFSET 0 ROWS FETCH NEXT 5 ROWS ONLY;

-- 5. Identify the longest movie

SELECT * FROM netflix_titles
WHERE
	type = 'Movie'
	AND
	duration = (SELECT MAX(duration) FROM netflix_titles)

-- 6. Find content added in the last 5 years

SELECT *
FROM netflix_titles
WHERE 
    CONVERT(DATE, date_added, 106) >= DATEADD(YEAR, -5, GETDATE());

-- 7. Find all the movies/TV shows by director 'Rajiv Chilaka'!

SELECT * FROM netflix_titles
WHERE director LIKE '%Rajiv Chilaka%'

-- 8. List all TV shows with more than 5 seasons

SELECT *
FROM netflix_titles
WHERE 
    TYPE = 'TV Show'
    AND 
    TRY_CAST(LEFT(duration, CHARINDEX(' ', duration) - 1) AS INT) > 5;

-- 9. Count the number of content items in each genre

WITH SplitGenres AS (
    SELECT 
        value AS genre
    FROM 
        netflix_titles
    CROSS APPLY 
        STRING_SPLIT(listed_in, ',')
)
SELECT 
    genre,
    COUNT(*) AS total_content
FROM 
    SplitGenres
GROUP BY 
    genre;

-- 10.Find each year and the average numbers of content release in India on netflix. return top 5 year with highest avg content release!

SELECT TOP 5
    country,
    release_year,
    COUNT(show_id) AS total_release,
    ROUND(
        CAST(COUNT(show_id) AS FLOAT) / 
        CAST((SELECT COUNT(show_id) FROM netflix_titles WHERE country = 'India') AS FLOAT) * 100, 
        2
    ) AS avg_release
FROM 
    netflix_titles
WHERE 
    country = 'India'
GROUP BY 
    country, release_year
ORDER BY 
    avg_release DESC

-- 11. List all movies that are documentaries

SELECT * FROM netflix_titles
WHERE listed_in LIKE '%Documentaries%'

--12. Find all content without a director

SELECT * FROM netflix_titles
WHERE  director IS NULL

-- 13. Find how many movies actor 'Salman Khan' appeared in last 10 years!

SELECT * 
FROM netflix_titles
WHERE cast LIKE '%Salman Khan%'
AND release_year > YEAR(GETDATE()) - 10;

-- 14. Find the top 10 actors who have appeared in the highest number of movies produced in India.


WITH ActorCTE AS (
    SELECT 
        TRIM(value) AS actor
    FROM 
        netflix_titles
    CROSS APPLY 
        STRING_SPLIT(cast, ',')
    WHERE 
        country = 'India'
)
SELECT 
    actor,
    COUNT(*) AS total_count
FROM 
    ActorCTE
GROUP BY 
    actor
ORDER BY 
    total_count DESC
OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY;

-- 15.Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing 
-- these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category.

SELECT 
    category,
    TYPE,
    COUNT(*) AS content_count
FROM (
    SELECT
        *,
        CASE 
            WHEN LOWER(description) LIKE '%kill%' OR LOWER(description) LIKE '%violence%' THEN 'Bad'
            ELSE 'Good'
        END AS category
    FROM netflix_titles
) AS categorized_content
GROUP BY 
    category, TYPE
ORDER BY 
    TYPE;

