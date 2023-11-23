
![OLYMPICS 1](https://github.com/Dparagon/olympics-history-analysis-1896--2016-with-sql/assets/128928568/63996564-c0af-4092-81f6-d7ee8c94c288)


### BACKGROUND

The modern Olympic Games were revived in 1896 in Athens by Pierre de Coubertin. Since that year, the event has grown into a global phenomenon bringing international unity through sport. The event occur every four (4) years, with both summer and winter editions showcasing wide range of sports and athletes all around the world.

### OVERVIEW AND OBJECTIVE

This project is about uncovering insights and analysing historical data related to Olympic Games from the year 1896 to 2016. The dataset is available on [kaggle](https://www.kaggle.com/datasets/heesoo37/120-years-of-olympic-history-athletes-and-results) and it has two tables namely: athlete and regions. The ‘athlete’ table consist of information about individual athlete participating in an event while ‘regions’ table consist of country information of athletes in the athlete table.

SQL Server was the tool used for this project and this gives me opportunity to develop my skills in SQL. These SQL skills were put into use for this project:
*	CTEs
*	Joins
*	Subqueries
*	Window Functions etc.
  
The tables were imported into SQL Server and were renamed for the purpose of this project. Athlete table renamed to ‘olympics_history’ and regions table renamed to ‘olympics_history_noc_regions’.

### ANALYSIS

The following insights were uncovered from the dataset as follows:
 
1.	How many Olympic Games have been held?
   
```sql
SELECT Season, COUNT(DISTINCT Year) AS No_of_games
FROM olympics_history
GROUP BY Season
```
![1](https://github.com/Dparagon/olympics-history-analysis-1896--2016-with-sql/assets/128928568/8a7e2294-c4a2-4ea0-ab15-956affa9249c)


2.	List all the Olympics held.

```sql
SELECT DISTINCT Year, Season
FROM olympics_history
```
![2](https://github.com/Dparagon/olympics-history-analysis-1896--2016-with-sql/assets/128928568/d9fd2319-18f5-49df-943b-2bd27e7d24b1)


3.	List the total no. of nations who participated in each Olympic Games.
   
```sql
SELECT Year, Season, COUNT(DISTINCT Team) AS No_of_nations
FROM olympics_history
GROUP BY Year, Season 
ORDER BY Year
```
![3](https://github.com/Dparagon/olympics-history-analysis-1896--2016-with-sql/assets/128928568/58ef0054-9ddc-4c91-8692-d3e9c5b5a380)


4.	Which year has the highest and lowest of nations participating in Olympics, and how many nations?

```sql
WITH oly_high AS (
SELECT TOP 1 Year, Season, COUNT(DISTINCT Team) AS nation
FROM olympics_history
GROUP BY Year, Season
ORDER BY nation DESC),

oly_low AS (
SELECT TOP 1 Year, Season, COUNT(DISTINCT Team) AS nation
FROM olympics_history
GROUP BY Year, Season
ORDER BY nation)
SELECT CONCAT(oly_high.Year,' ' , '-',' ' , oly_high.nation) AS highest_year,
       CONCAT(oly_low.Year,' ' , '-',' ' , oly_low.nation)  AS lowest_year
FROM oly_high, oly_low
```
![4](https://github.com/Dparagon/olympics-history-analysis-1896--2016-with-sql/assets/128928568/28fda840-8b48-421c-ac23-8a73dea4a34d)


5.	Which nation has participated in all of the Olympic Games?

```sql
SELECT Team, COUNT(Games) AS game_count
FROM olympics_history
GROUP BY Team
HAVING COUNT(Games) = (SELECT COUNT(DISTINCT Games)FROM olympics_history)
```
![5](https://github.com/Dparagon/olympics-history-analysis-1896--2016-with-sql/assets/128928568/66e1b4e2-167f-419b-9303-5d61804947c9)


6.	Which sport was played in all summer Olympic Games?

```sql
SELECT DISTINCT Sport 
FROM olympics_history
WHERE Games LIKE '%Summer'
```
![6](https://github.com/Dparagon/olympics-history-analysis-1896--2016-with-sql/assets/128928568/96205a3a-0244-455a-9ca1-e9397caa228c)


7.	Which sport was played once in the Olympics?

```sql
SELECT Sport, COUNT(DISTINCT Games) AS game_count
FROM olympics_history
GROUP BY Sport
HAVING COUNT(DISTINCT Games) = 1
```
![7](https://github.com/Dparagon/olympics-history-analysis-1896--2016-with-sql/assets/128928568/21035cee-09b4-47ae-bb3a-91cc97d86ba2)


8.	Give the total no. of sports played in each Olympic Games.

```sql
SELECT Games, COUNT(DISTINCT Sport) AS sport_count
FROM olympics_history
GROUP BY Games 
```
![8](https://github.com/Dparagon/olympics-history-analysis-1896--2016-with-sql/assets/128928568/80ffb449-1e34-4f32-bcf3-5c120514e1cf)


9.	Who are the oldest athletes to win a gold medal?

```sql
SELECT * 
FROM
   (SELECT Name, Age, Sex, Medal, Team, Games, DENSE_RANK() OVER(ORDER BY Age DESC) AS Age_rank
    FROM olympics_history
    WHERE medal = 'Gold' AND AGE <> 'NA') oly
WHERE Age_rank = 1
```
![9](https://github.com/Dparagon/olympics-history-analysis-1896--2016-with-sql/assets/128928568/1fc4d5bf-e025-4365-9f55-f3170ca5bc94)


10.	What is the ratio of male and female Athletes participation in all Olympics?

```sql
SELECT ROUND(CAST(SUM(CASE WHEN Sex = 'M' THEN 1 ELSE 0 END) AS FLOAT)/COUNT(*),2) AS Male_ratio,
       ROUND(CAST(SUM(CASE WHEN Sex = 'F' THEN 1 ELSE 0 END) AS FLOAT)/COUNT(*),2) AS Female_ratio
FROM 
    (SELECT DISTINCT Name, Sex FROM olympics_history) oly
```
![10](https://github.com/Dparagon/olympics-history-analysis-1896--2016-with-sql/assets/128928568/e7e1d6d8-54dc-4243-b1be-64694a0e21cb)


11.	Who are the top 5 Athletes with the highest medals?

```sql
SELECT TOP 5 Name, COUNT(Medal) AS Medal_count
FROM olympics_history
WHERE Medal <> 'NA'
GROUP BY Name
ORDER BY Medal_count DESC
```
![11](https://github.com/Dparagon/olympics-history-analysis-1896--2016-with-sql/assets/128928568/b615b10f-526e-4a30-9776-69da2ecb7a5b)


12.	List the total gold, silver and bronze medals won by each country.

```sql
WITH gold_medal AS (
           SELECT Team, COUNT(Medal) AS gold_count
           FROM olympics_history
           WHERE Medal = 'Gold'
		   GROUP BY Team ),
silver_medal AS (
          SELECT Team, COUNT(Medal) AS silver_count
          FROM olympics_history
          WHERE Medal = 'Silver'
		  GROUP BY Team ),
bronze_medal AS (
          SELECT Team, COUNT(Medal) AS bronze_count
          FROM olympics_history
          WHERE Medal = 'Bronze'
		  GROUP BY Team )
SELECT g.Team,
       g.gold_count,
	   s.silver_count,
	   b.bronze_count
FROM gold_medal g
JOIN silver_medal s ON g.Team = s.Team
JOIN bronze_medal b ON g.Team = b.Team
```
![12](https://github.com/Dparagon/olympics-history-analysis-1896--2016-with-sql/assets/128928568/5c12318b-b9bb-45a3-b4c0-280beab058d4)


13.	List the medals won by each country according to each Olympic Games.

```sql
WITH gold_medal AS (
           SELECT Games, Team, COUNT(Medal) AS gold_count
           FROM olympics_history
           WHERE Medal = 'Gold'
		   GROUP BY Games, Team ),
silver_medal AS (
          SELECT Games, Team, COUNT(Medal) AS silver_count
          FROM olympics_history
          WHERE Medal = 'Silver'
		  GROUP BY Games, Team ),
bronze_medal AS (
          SELECT Games, Team, COUNT(Medal) AS bronze_count
          FROM olympics_history
          WHERE Medal = 'Bronze'
		  GROUP BY Games,Team )
SELECT g.Games,
       g.Team,
       g.gold_count,
       s.silver_count,
       b.bronze_count
FROM gold_medal g
INNER JOIN silver_medal s ON g.Team = s.Team AND g.Games = s.Games
INNER JOIN bronze_medal b ON g.Team = b.Team AND g.Games = b.Games
ORDER BY g.Games ASC
```
![13](https://github.com/Dparagon/olympics-history-analysis-1896--2016-with-sql/assets/128928568/ede02a42-9b02-48f1-834e-e4de0b2ab4b3)


14.	Which country won the most gold, silver and bronze medals in each Olympic Games?

```sql
WITH gold_medal AS (
                   SELECT *, ROW_NUMBER() OVER(PARTITION BY Games ORDER BY gold_count DESC) gold_rank
				   FROM (SELECT Games, Team, COUNT(Medal) AS gold_count
				         FROM olympics_history
						 WHERE Medal = 'Gold'
						 GROUP BY Games, Team) AS gg ),
silver_medal AS (
                   SELECT *, ROW_NUMBER() OVER(PARTITION BY Games ORDER BY silver_count DESC) silver_rank
				   FROM (SELECT Games, Team, COUNT(Medal) AS silver_count
				         FROM olympics_history
						 WHERE Medal = 'Silver'
						 GROUP BY Games, Team) AS ss ),
bronze_medal AS (
                   SELECT *, ROW_NUMBER() OVER(PARTITION BY Games ORDER BY bronze_count DESC) bronze_rank
				   FROM (SELECT Games, Team, COUNT(Medal) AS bronze_count
				         FROM olympics_history
						 WHERE Medal = 'Bronze'
						 GROUP BY Games, Team) AS bb )
SELECT g.Games, 
         CONCAT(g.Team, ' - ' , g.gold_count) AS Highest_gold,
		 CONCAT(s.Team, ' - ' , s.silver_count) AS Highst_silver,
		 CONCAT(b.Team, ' - ' , b.bronze_count) AS Highest_bronze
FROM gold_medal AS g
JOIN silver_medal AS s ON g.Games = s.Games AND g.Team = s.Team
JOIN bronze_medal AS b ON g.Games = b.Games AND g.Team =s.Team
WHERE g.gold_rank = 1 AND s.silver_rank = 1 AND b.bronze_rank = 1
```
![14](https://github.com/Dparagon/olympics-history-analysis-1896--2016-with-sql/assets/128928568/e5e1622f-f65c-4583-9abf-2d5e44f41a10)


15.	Which country have never won gold medal but won silver medals and bronze medals?

```sql
SELECT * INTO olympic_details FROM (
                                    SELECT oh.*, hoc.region
                                    FROM olympics_history oh
                                    LEFT JOIN olympics_history_noc_regions hoc 
									ON oh.NOC = hoc.NOC) details

SELECT * FROM olympic_details


WITH gold_medal AS (
                SELECT DISTINCT region FROM olympic_details EXCEPT
				   SELECT region
				   FROM olympic_details
				   WHERE Medal = 'Gold'
				   GROUP BY region
				   HAVING COUNT(Medal) > 0 ),
silver_medal AS (
                   SELECT region, COUNT(Medal) AS silver_count
				   FROM olympic_details
				   WHERE Medal = 'Silver'
				   GROUP BY region  ),
bronze_medal AS ( 
                   SELECT region, COUNT(Medal) AS bronze_count
                   FROM olympic_details
				   WHERE Medal = 'Bronze'
				   GROUP BY region  )
SELECT g.region, 0 AS gold_count, s.silver_count, b.bronze_count
FROM gold_medal g 
LEFT JOIN silver_medal s ON g.region = s.region
LEFT JOIN bronze_medal b ON g.region = b.region
WHERE s.silver_count >= 1 OR b.bronze_count >= 1
ORDER BY g.region
```
![15](https://github.com/Dparagon/olympics-history-analysis-1896--2016-with-sql/assets/128928568/b82f24ed-9de1-4d77-aa7c-0a6969dff374)
