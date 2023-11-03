### BACKGROUND

The modern Olympic Games were revived in 1896 in Athens by Pierre de Coubertin. Since that year, the event has grown into a global phenomenon bringing international unity through sport. The event occur every four (4) years, with both summer and winter editions showcasing wide range of sports and athletes all around the world.

### OVERVIEW AND OBJECTIVE

This project is about uncovering insights and analysing historical data related to Olympic Games from the year 1896 to 2016. The dataset is available on kaggle and it has two tables namely: athlete and regions. The ‘athlete’ table consist of information about individual athlete participating in an event while ‘regions’ table consist of country information of athletes in the athlete table.
SQL Server was the tool used for this project and this gives me oppourtunity to develop my skills in SQL. These SQL skills were put into use for this project:
*	CTEs
*	Joins
*	Subqueries
*	Window functions etc.
  
The tables were imported into SQL Server and were renamed for the purpose of this project. Athlete table renamed to ‘olympics_history’ and regions table renamed to ‘olympics_history_noc_regions’.

### ANALYSIS

The following insights were uncovered from the dataset as follows:
 
1.	How many Olympic Games have been held?
   
```sql
SELECT Season, COUNT(DISTINCT Year) AS No_of_games
FROM olympics_history
GROUP BY Season
```

2.	List all the Olympics held.

```sql
SELECT DISTINCT Year, Season
FROM olympics_history
```

3.	List the total no. of nations who participated in each Olympic Games.
   
```sql
SELECT Year, Season, COUNT(DISTINCT Team) AS No_of_nations
FROM olympics_history
GROUP BY Year, Season 
ORDER BY Year
```

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

5.	Which nation has participated in all of the Olympic Games?

```sql
SELECT Team, COUNT(Games) AS game_count
FROM olympics_history
GROUP BY Team
HAVING COUNT(Games) = (SELECT COUNT(DISTINCT Games)FROM olympics_history)
```

6.	Which sport was played in all summer Olympic Games?

```sql
SELECT DISTINCT Sport 
FROM olympics_history
WHERE Games LIKE '%Summer'
```

7.	Which sport was played once in the Olympics?

```sql
SELECT Sport, COUNT(DISTINCT Games) AS game_count
FROM olympics_history
GROUP BY Sport
HAVING COUNT(DISTINCT Games) = 1
```

8.	Give the total no. of sports played in each Olympic Games.

```sql
SELECT Games, COUNT(DISTINCT Sport) AS sport_count
FROM olympics_history
GROUP BY Games 
```

9.	Who are the oldest athletes to win a gold medal?

```sql
SELECT * 
FROM
   (SELECT Name, Age, Sex, Medal, Team, Games, DENSE_RANK() OVER(ORDER BY Age DESC) AS Age_rank
    FROM olympics_history
    WHERE medal = 'Gold' AND AGE <> 'NA') oly
WHERE Age_rank = 1
```

10.	What is the ratio of male and female Athletes participation in all Olympics?

```sql
SELECT ROUND(CAST(SUM(CASE WHEN Sex = 'M' THEN 1 ELSE 0 END) AS FLOAT)/COUNT(*),2) AS Male_ratio,
       ROUND(CAST(SUM(CASE WHEN Sex = 'F' THEN 1 ELSE 0 END) AS FLOAT)/COUNT(*),2) AS Female_ratio
FROM 
    (SELECT DISTINCT Name, Sex FROM olympics_history) oly
```

11.	Who are the top 5 Athletes with the highest medals?

```sql
SELECT TOP 5 Name, COUNT(Medal) AS Medal_count
FROM olympics_history
WHERE Medal <> 'NA'
GROUP BY Name
ORDER BY Medal_count DESC
```

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
