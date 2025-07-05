![Netflix logo](https://github.com/aryan1jaiswal/Netflix_Movies_TvShows/blob/main/Netflix%20logo.jpg)

# Netflix Movies & TvShows (SQL+Powerbi)


## Overview

This project contains advanced SQL analysis and PowerBi Report Analysis on the **Netflix dataset**, which includes the following Columns:
- show_id
- type
- title 
- director
- casts  
- country 
- date_added
- release_year
- rating
- duration 
- listed_in (genres)
- description


## Objectives
- Identify content trends and shifts in genres, release patterns, and director popularity across years.
- Detect key actors, genres, and combinations that consistently co-occur or dominate viewership.
- Analyse country-level performance and uncover geographic content patterns and lags in availability.
- Measure content upload behaviour over time (monthly& yearly) and identify outlier months with spikes in uploads.
- Determine genre diversity, director impact, and the delay between release and Netflix availability.


## Schema

```sql
CREATE TABLE netflix (
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

<details> <summary><strong>Click to expand SQL Code</strong></summary>

### 1. For each type (Movie or TV Show), count the number of distinct genres across all titles. 

```sql

select type, COUNT(distinct genres)
from (
select type, TRIM(UNNEST(String_to_array(listed_in, ','))) as genres
from netflix
) abc
GROUP BY type;
```

### 2. Group titles by the month they were added (from date_added). Which month (across all years) has the highest average number of uploads?

```sql

select month_added, 
ROUND(COUNT(title)::numeric*100/ (select count(distinct title) from netflix)) as avg_uploads
from (
select *, DATE_TRUNC( 'Month', TO_DATE(date_added, 'Month DD, YYYY'))::date as month_added
from netflix
) abc
where month_added is not null
GROUP BY month_added
Order by  avg_uploads desc;

```

### 3. Find directors who have only one title on Netflix and that title was added before 2018. Return director name and title

```sql

with cte as (
select *, TRIM(UNNEST(STring_to_Array(director, ','))) as directors,
EXTRACT('Year' from TO_date(date_added, 'Month DD, YYYY')) as year
from netflix
)

select directors, count(distinct title) as title_cnt
from cte
where year <'2018'
group by directors
HAVING count(distinct title) = 1;
```


### 4. For each year, find the top 3 directors (by count of titles released). Return year, director, count, and their rank.

```sql

select *
from (
select *, Dense_Rank() Over (Partition BY year order by titles_cnt desc ) as rk
from (
select EXtract(Year from To_date(date_added, 'Month DD, YYYY')) as year,
TRIM(UNNEST(STRING_TO_ARRAY(director, ','))) as director,
COUNT(*) as titles_cnt
from netflix
GRoup by year, director
) abc
) abcd
where rk <=3

```

### 5. For each director, compute: Total number of distinct genres across all their titles (listed_in), Total number of titles. Then, return top 10 directors by genre diversity.

```sql

select director1, count(distinct title) as title_cnt, count(distinct genres) as genre_cnt
from (
select director1, title, TRIM(UNNEST(String_to_array(listed_in, ','))) as genres
from (
select *, TRIM(UNNEST(String_to_array(director, ','))) as director1
from netflix
) abc
where TRIM(director1) <> ''
) abcd
GROUP BY director1
ORDER BY genre_cnt desc
LIMIT 10;
```

### 6. Top 10 Actors Who Have Appeared in the Highest Number of Movies Produced in India

```sql

SELECT UNNEST(STRING_TO_ARRAY(casts, ',')) AS actor,
COUNT(*) as cnt
FROM netflix
WHERE country = 'India'
GROUP BY actor
ORDER BY cnt DESC
LIMIT 10;

```


### 7. Find actors who appeared in 5 or more titles across at least 3 different genres (based on listed_in). Return actor name, number of unique genres, and total title count.

```sql

with actor_data as (
select *, TRIM(UNNEST(String_to_Array(casts, ','))) as actors
from netflix
), 
genre_data as (
select *, TRIM(UNNEST(String_to_Array(listed_in, ','))) as genres
from actor_data 
)

select actors, COUNT(distinct title) as titles_cnt, COUNT(distinct genres) as genre_cnt
from genre_data
Group by actors
HAVING COUNT(distinct genres) >= 3 and COUNT(distinct title) >= 5

```

###  8. For each country, calculate how many titles were released each year. Then identify countries where the number of titles released decreased from 2020 to 2021.

```sql

with countries as (
select country1, year,
(CASE WHen year = '2020' Then titles_cnt ELSE 0 end) AS title_cnt1,
(CASE WHen year = '2021' Then titles_cnt ELSE 0 end) AS title_cnt2
from 
(
select TRIM(UNNEST(STRING_TO_ARRAY(country, ','))) as country1, 
EXTRACT('YEAR' from TO_DATE(date_added, 'Month DD, YYYY')) as year,
COUNT(*) as titles_cnt
from netflix
where date_added is not null and country is not null
GROUP BY country1, year
ORDER BY country1, year
) abc
where TRIM(country1) <> ''
AND (year = '2020' OR year = '2021')
)

select country1, SUM(title_cnt1) as titles_in_2020, SUM(title_cnt2) as titles_in_2021
from countries
Group By country1
HAVING SUM(title_cnt1) > SUM(title_cnt2)

```

### 9. Find the Most Common Rating for Movies and TV Shows

```sql

#solution 1

select a.rating, a.c1, b.c2
from (
select rating, count(*) as c1
from netflix
where type = 'Movie'
group by rating
) a 
JOIN (
select rating, count(*) as c2
from netflix
where type = 'TV Show'
group by rating
) b
ON a.rating = b.rating
order by a.c1 desc 
limit 1; 

#solution 2

(
select rating, count(*) as c1
from netflix
where type = 'Movie'
group by rating
order by c1 desc 
limit 1
) 
UNION ALL
(
select rating, count(*) as c2
from netflix
where type = 'TV Show'
group by rating
order by c2 desc 
limit 1
) 
```


### 10. For each genre, calculate the total number of titles released per year. Then, identify genres where yearly output has increased every year since 2018.

```sql

with genre_data as (
select *, 
EXtract(Year from To_date(date_added, 'Month DD, YYYY')) as year, 
TRIM(UNNEST(String_to_Array(listed_in, ','))) as genres
from netflix
),
yearly_genre_data as (
select genres, year, COUNT(distinct title) as titles_cnt
from genre_data
where year is not null 
and year>= '2018'
Group by genres, year
), 
genre_comparison as (
select *, LAG(titles_cnt) Over(Partition By genres Order by year) as prev_cnt
from yearly_genre_data
),
bad_genres as (
select distinct genres
from genre_comparison
where prev_cnt > titles_cnt
)

select *
from yearly_genre_data 
where genres NOT IN
(Select * from bad_genres)

```

### 11. Find the country with the longest streak of consecutive years having at least one release. Return the country name and length of the streak.

```sql

with grouped_country as (
select *, year - Interval '1 year' * (rk-1) as grp_key
from 
(
select distinct country1, year, Dense_RAnk() Over (Partition BY country1 Order by year) as rk
from (
select *, TRIM(UNNEST(STRING_TO_ARRAY(country, ','))) as country1,
DATE_TRUNC('Year', TO_date(date_added, 'Month DD, YYYY'))::date as year
from netflix
) abc
where TRIM(country1) <> '' AND year is not null
order by country1, year
)
),
streak_countries as (
select country1, grp_key, COUNT(*) as streak_length
from grouped_country
GROUP BY country1, grp_key
)

select country1, streak_length
from streak_countries
order by streak_length desc
limit 1;

```


### 12. For each title, calculate the number of days between release_year and date_added. Find the average lag per country, and list the top 5 countries with the longest average delay in uploading content.

```sql

with date_info as (
select country, title,
TO_date(date_added, 'Month DD, YYYY') as date_added,
TO_date(release_year::varchar, 'YYYY') as release_date
from netflix
),
lag_info as (
select TRIM(UNNEST(String_to_array(country, ','))) as country, title, 
(release_date - date_added) as lag_time
from date_info
where country is not null
)

select country, ROUND(AVG(lag_time)) as avg_lag
from lag_info
where TRIM(country) <> ''
Group by country
order by AVG(lag_time)
Limit 5;
```


### 13. From the listed_in column, extract pairs of genres (like "Action" & "Drama") that co-occur together in at least 50 titles. Return each pair and their count.

```sql

with genre_unnest as (
select title, TRIM(UNNEST(String_to_array(listed_in, ','))) as genres
from netflix
where listed_in is not null
),
paired as (
select a.title,
a.genres as genre1, b.genres as genre2
from genre_unnest a 
JOIN genre_unnest b
ON a.title = b.title
AND LENGTH(a.genres) < LENGTH(b.genres)
)

select genre1, genre2, COUNT(distinct title) as title_cnt
from paired
GROUP BY genre1, genre2
HAVING COUNT(distinct title) >= 50
order by title_cnt;
```


### 14. For each title, compute the delay in days between its release year and the date it was added.Then return the top 1% of titles with the longest delay.

```sql

with date_info as (
select title,
TO_date(date_added, 'Month DD, YYYY') as date_added,
TO_date(release_year::varchar, 'YYYY') as release_date
from netflix
), 
delay_days as (
select distinct title, 
(date_added - release_date) as delay_time
from date_info
where date_added >= release_date
),
ranked_delay as (
select *,  PERCENT_RANK() OVER (ORDER BY delay_time) AS percentile
from delay_days
where delay_time is not null
)

select *
from ranked_delay
where percentile >= 0.99
Order by delay_time desc;

```

### 15. For each month in each year, calculate the number of uploads. Then identify months where: Uploads are at least twice the average monthly uploads of that year.

```sql

with monthly_uploads as (
select month_added, year_added, count(distinct title) as uploads
from (
select title, EXTRACT('Month' from TO_DATE(date_added, 'Month DD, YYYY')) as month_added,
EXTRACT('Year' from TO_DATE(date_added, 'Month DD, YYYY')) as year_added
from netflix
where date_added is not null
)
GROUP BY month_added, year_added
), 

year_data as (
select year_added, count(distinct title)::numeric/12 as monthly_average
from (
select title, 
EXTRACT('Year' from TO_DATE(date_added, 'Month DD, YYYY')) as year_added
from netflix
where date_added is not null
) abc
GROUP BY year_added
)

select a.*, b.monthly_average
from 
monthly_uploads a JOIN year_data b
ON a.year_added = b.year_added
AND a.uploads >= 2* b.monthly_average
```
