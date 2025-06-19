 # 📺 Netflix Data Analysis with SQL

This project explores Netflix's content dataset using **SQL** (PostgreSQL/MySQL compatible) to uncover **15 real-world business insights**. It demonstrates data wrangling, aggregation, and analytical querying on a structured dataset.

---

## 📂 Dataset

- Source: [`netflix_titles.csv`](https://www.kaggle.com/datasets/shivamb/netflix-shows)
- Fields include:  
  `show_id`, `type`, `title`, `director`, `cast`, `country`, `date_added`, `release_year`, `rating`, `duration`, `listed_in`, `description`

---

## 🛠️ Setup

1. **Create the database**:
    ```sql
    CREATE DATABASE netflix_db;
    ```

2. **Create the table**:
    ```sql
    CREATE TABLE netflix_data (
      show_id VARCHAR(10),
      types VARCHAR(10),
      title VARCHAR(150),
      director VARCHAR(255),
      casts VARCHAR(1500),
      country VARCHAR(150),
      date_added VARCHAR(50),
      release_year INT,
      rating VARCHAR(10),
      duration VARCHAR(50),
      listed_in VARCHAR(100),
      descriptions VARCHAR(250)
    );
    ```

3. **Import the data**:
    *(Uncomment and update path accordingly)*  
    ```sql
    -- LOAD DATA LOCAL INFILE 'path_to/netflix_titles.csv'
    -- INTO TABLE netflix_data
    -- FIELDS TERMINATED BY ','
    -- ENCLOSED BY '"'
    -- LINES TERMINATED BY '\r\n'
    -- IGNORE 1 ROWS;
    ```

---

## 🔍 Business Problems & SQL Solutions

1. **Content Type Distribution**  
   Count the number of Movies vs TV Shows
   ```sql
    select types,
    count(*) as total_content
    from netflix_data
    group by types;
   ```

2. **Most Common Rating by Type**  
   Identify dominant ratings in each category
   ```sql
   select types,rating
	from (
        select types, rating,
        count(*), 
        rank() over(partition by types order by count(*) desc ) as ranking 
        from netflix_data
        group by 1, 2
    ) as t1
    where ranking = 1;
   ```

3. **List all movies released in a specific year (e.g., 2020)**  
   Movies Released in 2020
   ```sql
    select * from netflix_data where 
	types = 'Movie'
    and 
    release_year = 2020;
   ```

4. **Find the top 5 countries with the most content on Netflix**  
   Normalize multivalued fields and rank
   ```sql
    select 
	unnest(string_to_array(country, ',')) as new_country,
    count(show_id)  as total_content
    from netflix_data 
    group by 1
    order by 2 desc
    limit 5;
   ```

5. **Identify the longest movie or TV show duration**  
   Find maximum duration entry
   ```sql
    select * from netflix_data
	where
    types = 'Movie'
    and 
    duration = (select max(duration) from netflix_data);
   ```

6. **Find content added in the last 5 years** *(optional)*  
   Parse dates and filter
   ```sql
    select * from netflix_data
	where 
    to_date(date_added, 'month DD YYYY') >= current_date - interval '5 years';
   ```

7. **Find all the movies/TV shows by director 'Rajiv Chilaka'**  
   Query content by Rajiv Chilaka
   ```sql
   select * from netflix_data where director like '%Rajiv chilaka%';
   ```

8. **List all TV shows with more than 5 seasons**  
   Extract season count from duration
   ```sql
    select * from netflix_data 
    where 
    type = 'TV show'
    and 
    SPLIT_PART(duration, ' ', 1) :: numeric >5 ;
   ```

9. **Count the number of content items in each genre**  
   Unnest multigenre entries
   ```sql
   select unnest(string_to_array(listed_in, ',',)) as genre,
   count(show_id) as total_content
   from netflix_data
   group by 1;
   ```

10. **Find each year and the average numbers of content release by India on netflix. return top 5 year with highest avg content release !**  
   Compute yearly average content release
   ```sql
    SELECT
    EXTRACT(YEAR FROM TO_DATE(date_added, 'Month DD, YYYY')) as year,
    COUNT(*) as yearly_content,
    ROUND (
    COUNT(*):: numeric/(SELECT COUNT(*) FROM netflix WHERE country ='India'):: numeric * 100, 2)as avg_content_per_year
    FROM netflix
    WHERE country = 'India'
    GROUP BY 1;
   ```

11. **List all movies that are documentaries**  
   Filter by keywords in genre
   ```sql
    select * from  netflix_data where listed_in like '%documentaries%';
   ```
    
12. **Find all content without a director**  
   Identify entries without directors
   ```sql
   select * from netflix_data where director is null;
   ```

13. **Find how many movies actor 'Salman Khan' appeared in last 10 years!**  
   Find all recent titles with this actor
   ```sql
   SELECT * FROM netflix_data
 WHERE casts LIKE '%Salman Khan%'
 AND release_year > EXTRACT (YEAR FROM CURRENT_DATE) - 10;
   ```

14. **Find the top 10 actors who have appeared in the highest number of movies produced in India.**  
   Rank based on appearance count
   ```sql
   SELECT
UNNEST(STRING_TO_ARRAY(casts, ',')) as actors,
COUNT(*) as total_content
FROM netflix_data
WHERE country ILIKE '%india%'
GROUP BY 1
ORDER BY 2 DESC	
LIMIT 10;
   ```

15. **Categorize the content based on the presence of the keywords 'kill' and 'violence' in the description field. Label content containing these keywords as 'Bad' and all other content as 'Good'. Count how many items fall into each category.**  
   Use description to label as *Good* or *Bad*
   ```sql
    WITH new_table
    AS (
    SELECT *,
	    CASE
	    WHEN
	        description ILIKE '%kill%' OR
	        description ILIKE '%violence%' THEN 'Bad_Content'
	        ELSE 'Good Content'
	        END category
	    FROM netflix) 
    SELECT category,
	    COUNT(*) as total_content
	    FROM new_table
	    GROUP BY 1;
   ```

---

## 🧠 Tech Stack

- SQL (PostgreSQL / MySQL)
- Data Cleaning & Analysis
- Window Functions, Aggregation, CTEs, String Processing

---

## 📌 Note

- Use appropriate SQL dialect for your DBMS (PostgreSQL recommended for string/array handling).
- Date parsing and advanced string manipulation may vary slightly across MySQL and PostgreSQL.

---

## ⭐️ If you find this helpful, please give the repo a star!


### 🔧 Installation & Usage- Clone the repository:
git clone https://github.com/siddhesh-appzlogic/SQL_Netflix_data_analysis.git
- Run the SQL queries for data exploration.

## 🏷️ License
This project is licensed under the MIT License.

## 🧑‍💻 Author

**Mr. Siddhesh Walunj**  
GitHub: [https://github.com/SiddheshWalunj](https://github.com/siddhesh-appzlogic)

## 🤝 Contributions
Feel free to contribute by submitting pull requests or suggesting improvements!
