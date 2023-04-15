---
title: Homework "#1 - SQL"
categories: SQL
tags: sql
date: 2023-04-13
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
<font color=red size=5>暂时写了这么多</font>
