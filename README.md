# Netflix Movies and TV Shows Data Analysis using SQL 
![Netflix logo](https://github.com/em1407/SQL-Netflix-Project/blob/main/Netflix-Logo.png)

    DROP TABLE IF EXISTS netflix;
    CREATE TABLE netflix
    (
    	show_id	VARCHAR(6),
    	type	VARCHAR (10),
    	title	VARCHAR (150),
    	director	VARCHAR (208),
    	casts	VARCHAR (1000),
    	country	VARCHAR (150),
    	date_added	VARCHAR (50),
    	release_year INT,
    	rating		VARCHAR (10),
    	duration	VARCHAR (15),
    	listed_in	VARCHAR (100),
    	description	VARCHAR (250)

    );
    
    SELECT * FROM netflix;
    
    SELECT 
    	COUNT(*) as total_content
    FROM netflix;



    SELECT 
    	DISTINCT type
    FROM netflix;

## 15 Business Problems

## 1. Count the number of Movies vs TV Shows
    SELECT type,
    	COUNT(*) as total_content
    FROM netflix
    GROUP BY type;
    
## 2.Find the most common rating for Movies and TV Shows 
    SELECT type, rating
    FROM
    (
    	SELECT type, rating,
    		COUNT(*),
    		RANK() OVER(PARTITION BY type ORDER BY COUNT(*) DESC) as ranking
    	FROM netflix
    	GROUP BY 1,2
    ) as t1
    WHERE ranking = 1


## 3. List all nmovies released in a specific year (e.g. 2020) 
    SELECT *
    FROM netflix
    WHERE type = 'Movie' AND release_year = 2020


## 4. Find the top 5 countries with the most content on Netflix

    SELECT 
    	UNNEST(STRING_TO_ARRAY(country, ',')) as new_country,
    	COUNT(show_id) as total_content
    FROM netflix
    GROUP BY country
    ORDER BY 2 DESC
    LIMIT 5

## 5. Identify the longest Movie or TV Show duration 
    SELECT * FROM netflix
    WHERE 
    	type = 'Movie'
    	AND 
    	duration = (SELECT MAX(duration) FROM netflix)


## 6. Find content added in the last 5 years 
    SELECT 
    	*
    FROM netflix
    WHERE 
    	TO_DATE(date_added, 'Month DD,YYYY') >= CURRENT_DATE - INTERVAL '5 years'


## 7. Find all the movies/TV Shows by director 'Rajiv Chilaka'

    SELECT 
    	*
    FROM netflix
    WHERE director LIKE '%Rajiv Chilaka%'


## 8. List all TV Shows with more than 5 seasons
    SELECT 
    	*
    FROM netflix
    WHERE type = 'TV Show' 
    AND
    SPLIT_PART(duration, ' ', 1):: numeric > 5 


## 9. Count the number of content items in each genre
    SELECT 
    	UNNEST(STRING_TO_ARRAY(listed_in, ',')) as unique_genre,
    	COUNT(title)
    FROM netflix
    GROUP BY unique_genre


## 10. Find each year and the average numbers of content released by India on Netlix. Return the top 5 with highest avg content release 
    SELECT 
    	EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) as year, 
     	COUNT(*) AS yearly_content,
    	 ROUND(
    	COUNT(*)::numeric/(SELECT COUNT(*) FROM netflix WHERE country = 'India') :: numeric * 100 
    	,2)as avg_content_per_year
    FROM netflix
    WHERE country = 'India'
    GROUP BY 1



## 11. List all movies that are documentaries
    SELECT 
    	 *
    FROM netflix
    WHERE listed_in ILIKE '%Documentaries%'

## 12. Find all content without a director
    SELECT 
    	* FROM netflix
    WHERE director isnull

## 13. Find in how many movies the actor 'Salman Khan' appeared in the last 10 year
    SELECT *
    FROM netflix
    WHERE 
    	casts ILIKE '%Salman Khan%'
    	AND
    	release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10

## 14. Find the top 10 actors who have appeared in the highest number of movies produced in India
    SELECT
    	UNNEST(STRING_TO_ARRAY(casts, ',')) as unique_actor,
    	COUNT(*) AS total_content
    FROM netflix
    WHERE country ILIKE 'India'
    GROUP BY 1
    ORDER BY 2 DESC
    LIMIT 10

## 15. Categorize the content based on the presence of keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category
    WITH new_table
    AS(
    SELECT 
    	*,
    	CASE
    	WHEN description ILIKE '%kill%' 
    	OR 
    	description ILIKE '%violence%' THEN 'Bad Content'
    	ELSE 'Good Content'
    	END category
    FROM netflix
    )
    SELECT 
    	category,
    	COUNT(*) AS total_content
    	FROM new_table
    	GROUP BY 1
