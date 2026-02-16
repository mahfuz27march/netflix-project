# Netflix-project data analysis with Postgresql
![Netflix Logo](https://github.com/mahfuz27march/netflix-project/blob/main/logo.png)

## Objective

```table creating
drop table if exists netflix;
create table netflix(
	show_id varchar(10),
	type varchar(10),
	title varchar(150),
	director varchar(250),
	casts varchar(1000),
	country varchar(150),
	date_added varchar(50),
	release_year int,
	rating varchar(10),
	duration varchar(15),
	listed_in varchar(100),
	description varchar(250)
	);
select count(*) from netflix	
select distinct type from netflix
```
----------------------------------------
-- problems solving-------------
#1. Count the Number of Movies vs TV Shows

```select type,
	count(*)
	from netflix
group by 1
```
-----------------
#2. Find the Most Common Rating for Movies and TV Shows

```select type,rating from
	(select 
		type,
		rating,
		count(*),
		rank() over(partition by type order by count(*) desc) as ranking
	from netflix
	group by 1,2
	) as t1
where ranking=1	
--order by 1,3 desc
```
-------------------------------------------------------

#3. List All Movies Released in a Specific Year (e.g., 2020)

```select * from netflix
where release_year=2020 and type='Movie'
-------------------------------------------------------
```
#4. Find the Top 5 Countries with the Most Content on Netflix

```select 
	unnest(string_to_array(country,',')) as new_country,
	count(show_id) as total_content
from netflix
group by 1 
order by 2 desc
limit 5
--------------------------------------------------------
```
#5. Identify the Longest Movie

```select * from netflix
	where type='Movie'
	and
	duration=(select max(duration)from netflix)
-----------
SELECT 
    *
FROM netflix
WHERE type = 'Movie'
ORDER BY SPLIT_PART(duration, ' ', 1)::INT DESC;
---------------------------------------------------------```
#6. Find Content Added in the Last 5 Years

```select count(*) from netflix
	where to_date(date_added,'month dd,yyyy')>=current_date - interval '5 years'
---------------------------------------------------------```
#7. Find All Movies/TV Shows by Director 'Rajiv Chilaka'

```select * from (
	select *, unnest(string_to_array(director, ',')) as direktor_name from netflix
	) as t1
where direktor_name='Rajiv Chilaka'	
----alternate way
select * from netflix
where director like '%Rajiv Chilaka%'
---------------------------------------------------------
```
#8. List All TV Shows with More Than 5 Seasons

```select * from netflix
	where type='TV Show' 
	and split_part(duration,' ',1)::int>5
---------------------------------------------------------
```
#9. Count the Number of Content Items in Each Genre

```select
count(show_id),
unnest(string_to_array(listed_in,',')) as genre
from netflix
group by 2
---------------------------------------------------------
```
#10.Find each year and the average numbers of content release in India on netflix.

```select 
	extract(year from to_date(date_added,'month dd,yyyy')) as year,
	count(*) as total_content,
	Round(
		count(*)::numeric/(select count(*) from netflix where country='India')::numeric*100
		,2) as avg_content
from netflix
where country='India'
group by 1
---------------------------------------------------------
```
#11. List All Movies that are Documentaries

```select * from netflix
where listed_in like '%Documentaries%' and type='Movie'
-------
SELECT * 
FROM netflix
WHERE listed_in LIKE '%Documentaries';
----------------------------------------------------------
```
#12. Find All Content Without a Director
```select * from netflix
where director is null
----------------------------------------------------------
```
#13. Find How Many Movies Actor 'Salman Khan' Appeared in the Last 10 Years

```select
	count(*) 
from netflix
where casts like '%Salman Khan%' 
	and
	release_year>extract(year from current_date) -10
--------
SELECT * 
FROM netflix
WHERE casts LIKE '%Salman Khan%'
  AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10;
 ----------------------------------------------------------- 
```
#14. Find the Top 10 Actors Who Have Appeared in the Highest Number of Movies
 --Produced in India

 ```select * from netflix
 select 
 	--show_id,
	unnest(string_to_array(casts,','))as actors,
	count(*)
from netflix
where type='Movie' and country='India'
group by 1
order by 2 desc
limit 10
-----------------------------------------------------------
```
#15. Categorize Content Based on the Presence of 'Kill' and 'Violence' Keywords
--Objective: Categorize content as 'Bad' if it contains 'kill' or 'violence' and 
--'Good' otherwise. Count the number of items in each category.

```with new_table
as(
select 	
	case
		when description ilike '%kill%'
		or description ilike '%violence%' then 'Bad'
		else 'Good'
	end	category
from netflix
)
select 
	category,
	count(*) 
from new_table
group by 1
-------------------------------------------------------------
```
