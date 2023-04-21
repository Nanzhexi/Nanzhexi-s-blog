---
title: Homework "#1 - SQL"
categories: SQL
tags: sql
date: 2023-04-21
---
```
sqlite> select count(*) from titles;
1375462
```
![ig18ux.png](https://i.328888.xyz/2023/04/13/ig18ux.png)

**Hour Countdown 构造SQL查询 （作业部分）**
作业官网有公布答案，请自行参阅
建议用菜鸟教程来熟悉SQLite3
<font size=5>**Q1：[0 points] (q1_sample)**</font>
**Brief:**
The purpose of this query is to make sure that the formatting of your output matches exactly the formatting of our auto-grading script.

**Details:** 
List all Category Names ordered alphabetically.
只是一个测试问题。确保格式正确。
Solutions
用vim在placeholder的q1_sample.sql文件，写上下列命令来测试：
```
		sqlite> SELECT DISTINCT(language)
   		...> FROM akas
   		...> ORDER BY language
   		...> LIMIT 10;
```
<font size=5>**Q2：[5 points] (q2_sci_fi)**</font>

**Brief:** 
Find the 10 Sci-Fi works with the longest runtimes.

**Details:** 
Print the title of the work, the premiere date, and the runtime. The column listing the runtime should be suffixed with the string " (mins)", for example, if the <font color=red>runtime_mins</font> value is 12, you should output <font color=red>12 (mins)</font>. Note a work is <font color=red>Sci-Fi</font> even if it is categorized in multiple genres, as long as <font color=red>Sci-Fi</font> is one of the genres.
Your first row should look like this: <font color=red>Cicak-Man 2: Planet Hitam|2008|999 (mins)</font>
	Solutions
```
compound value of SELECT primary_title, premiered, runtime_minutes || ' (mins)'
FROM titles WHERE genres LIKE '%Sci-Fi%'
ORDER BY runtime_minutes DESC
LIMIT 10;
```
<font size=5>**Q3: [5 points] (q3_oldest_people)**</font>
**Brief:** 
Determine the oldest people in the dataset who were born in or after 1900. You should assume that a person without a known death year is still alive.

**Details:** 
Print the name and age of each person. People should be ordered by a compound value of their age and secondly their name in alphabetical order. Return the first 20 results.
Your output should have the format: <font color=red>NAME|AGE</font>
	Solutions
```
SELECT name,age
FROM (
    SELECT name, (2022 - born) AS age
    FROM people
    WHERE born >=1900 AND died IS NULL
    UNION
    SELECT name, (died - born) AS age
    FROM people
    WHERE born >=1900 AND died IS NOT NULL
    )
ORDER BY age DESC,name
LIMIT 20;
```
<font size=5>**Q4: [10 points] (q4_crew_appears_most)**</font>
**Brief:** 
Find the people who appear most frequently as crew members.

**Details:** 
Print the names and number of appearances of the 20 people with the most crew appearances ordered by their number of appearances in a descending fashion.
Your output should look like this: <font color=red>NAME|NUM_APPEARANCES</font>
```
SELECT name, 
		COUNT(*) as num_appearances
FROM people INNER JOIN crew Solutions:
	ON people.person_id = crew.person_id
GROUP BY name
ORDER BY num_appearances DESC
LIMIT 20;
```
<font size=5>**Q5: [10 points] (q5_decade_ratings)**</font>
**Brief:**
Compute intersting statistics on the ratings of content on a per-decade basis.

**Details:**
Get the average rating (rounded to two decimal places), top rating, min rating, and the number of releases in each decade. Exclude titles which have not been premiered (i.e. where premiered is NULL). Print the relevant decade in a fancier format by constructing a string that looks like this: 1990s. Order the decades first by their average rating in a descending fashion and secondly by the decade, ascending, to break ties.
Your output should have the format: <font color=red>DECADE|AVG_RATING|TOP_RATING|MIN_RATING|NUM_RELEASES</font>
```
SELECT
	CAST(permiered/10 * 10 AS TEXT) || 's' AS decade, 
	ROUND(AVG(rating), 2) AS avg_rating,
	MAX(rating) AS top_rating,
	MIN(rating) AS min_rating,
	COUNT(*) AS num_releases
FROM titles INNER JOIN ratings
	ON titles.title_id = ratings.title_id
WHERE premiered IS NOT NULL
GROUP BY decade
ORDER BY avg_rating DESC, decade ASC
```
<font size=5>**Q6: [10 points] (q6_cruiseing_altitude)**</font>
**Brief:**
Determine the most popular works with a person who has "Cruise" in their name and is born in 1962.

**Details:**
Get the works with the most votes that have a person in the crew with "Cruise" in their name who was born in 1962. Return both the name of the work and the number of votes and only list the top 10 results in order from most to least votes. Make sure your output is formatted as follows: <font color=red>Top Gun|408389</font>
```
select primary_title,votes from titles,ratings,people,crew 
   	where name like "%Cruise%"
  	and born=1962
   	and people.person_id=crew.person_id
   	and crew.title_id=titles.title_id
   	and crew.title_id=ratings.title_id
   	order by votes desc
   	limit 10;
```
**或**
```
select primary_title,votes from crew 
        inner join people on people.person_id=crew.person_id
   	inner join ratings on ratings.title_id=crew.title_id
   	inner join titles on titles.title_id=crew.title_id
   	where name like "%Cruise%"
  	order by votes desc
   	limit 10;
```
<font size=5>**Q7: [15 points] (q7_year_of_thieves)**</font>
**Brief:**
List the number of works that premiered in the same year that "Army of Thieves" premiered.

**Details:**
Print only the total number of works. The answer should include "Army of Thieves" itself. For this question, determine distinct works by their <font color=red>title_id</font>, not their names.
```
select count(distinct title_id) from titles
   	where premiered in (select premiered from titles where primary_title="Army of Thieves");
```
<font size=5>**Q8: [15 points] (q8_kidman_colleagues)**</font>
**Brief:**
List the all the different actors and actresses who have starred in a work with Nicole Kidman (born in 1967).

**Details:**
Print only the names of the actors and actresses in alphabetical order. The answer should include Nicole Kidman herself. Each name should only appear once in the output.

**Note:** As mentioned in the schema, when considering the role of an individual on the crew, refer to the field <font color=red>category</font>. The roles "actor" and "actress" are different and should be accounted for as such.
```
with Nice_work as ( select distinct (crew.title_id) 
 	from people 
 	inner join crew on people.person_id=crew.person_id
 	and people.name="Nicole Kidman"
 	and people.born=1967),
 	Nico_colleagues as ( select distinct (crew.person_id) as id
 	from crew
 	where (crew.category ="actor" or crew.category = "actress") and crew.title_id in Nice_work )
 	select name
 	from people join Nico_colleagues on Nico_colleagues.id = people.person_id
 	order by name desc;
```
<font size=5>**Q9: [15 points] (q9_9th_decile_ratings)**</font>
**Brief:**
For all people born in 1955, get their name and average rating on all movies they have been part of through their careers. Output the 9th decile of individuals as measured by their average career movie rating.

**Details:**
Calculate average ratings for each individual born in 1955 across only the movies they have been part of. Compute the quantiles for each individual's average rating using NTILE(10).
Make sure your output is formatted as follows (round average rating to the nearest hundredth, results should be ordered by a compound value of their ratings descending and secondly their name in alphabetical order): <font color=red>Stanley Nelson|7.13</font>

**Note:** You should take quantiles after processing the average career movie rating of individuals. In other words, find the individuals who have an average career movie rating in the 9th decile of all individuals.
```
WITH actor_movie_1955 AS (
	SELECT
    	people.person_id,	
    	people.name,
    	titles.title_id	
    FROM
    	people
    INNER JOIN
    	crew ON people.person_id = crew.person_id
    INNER JOIN
    	titles ON titles.title_id = crew.title_id
    WHERE people.born = 1955 AND titles.type = "movie"
),	
actor_ratings AS (
	SELECT
    	name,
    	ROUND(AVG(ratings.rating), 2) AS rating
    FROM ratings
    INNER JOIN actor_movie_1955
    	ON ratings.title_id = actor_movie_1955.title_id
    GROUP BY actor_movie_1955.person_id
),
quartiles AS (
	SELECT *, NTILE(10) OVER (ORDER BY rating ASC)
    	AS RatingQuartile FROM actor_ratings
)
SELECT name, rating
FROM quartiles
WHERE RatingQuartile = 9
ORDER BY rating DESC, name ASC;
```
<font size=5 color=red>**q10 有点难,容我再慢慢想**</font>
