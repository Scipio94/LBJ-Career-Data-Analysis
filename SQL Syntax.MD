
SQL Syntax for the queries ran during the analysis of the dataset. Corresponding query results will be uploaded as seperate files.  

1. Joining the two datasets using the **game_id** column

~~~ SQL

/* JOINING Datasets to return LeBron's stats from both datasets*/

SELECT *
 FROM `my-data-project-36654.NBA_Data.NBA_game_detail` AS gd 
 JOIN `my-data-project-36654.NBA_Data.NBA_game_dataset` AS g USING (game_id)
 WHERE PLAYER_NAME = 'LeBron James';
~~~~


2. LeBron James career stats by team

~~~ SQL
SELECT 
    DISTINCT PLAYER_NAME,TEAM_ABBREVIATION,COUNT (DISTINCT SEASON) OVER (PARTITION BY TEAM_ABBREVIATION) AS SeasonCount,
    COUNT(DISTINCT SEASON) OVER () AS SeasonTotal,COUNT (GAME_ID) OVER (PARTITION BY TEAM_ABBREVIATION) AS GameCount, 
    COUNT (GAME_ID) OVER () AS TotalGames, SUM(PTS) OVER (PARTITION BY TEAM_ABBREVIATION) AS TeamPoints, 
    SUM(FG3M) OVER (PARTITION BY TEAM_ABBREVIATION) * 3 AS Team_PTS_FG3, SUM(FG3M) OVER () * 3 AS PTS_FG3_Total,
    SUM(FGM-FG3M) OVER (PARTITION BY TEAM_ABBREVIATION) * 2 AS Team_PTS_FG2M,SUM(FGM-FG3M) OVER () * 2 AS PTS_FG2_Total,
    SUM(FTM) OVER (PARTITION BY TEAM_ABBREVIATION) AS Team_PTS_FTM,SUM(FTM) OVER () AS PTS_FT_Total,
    SUM(PTS) OVER () AS CareerPoints
FROM `my-data-project-36654.NBA_Data.NBA_game_detail` AS gd 
JOIN `my-data-project-36654.NBA_Data.NBA_game_dataset` AS g USING (game_id)
WHERE PLAYER_NAME = 'LeBron James'
ORDER BY SeasonCount;
~~~


3. Career Trend of LeBron James based on shot selection
~~~ SQL

/* Trends*/
WITH Trends AS (
SELECT 
   DISTINCT SEASON, PLAYER_NAME,TEAM_ABBREVIATION, COUNT (GAME_ID) OVER (PARTITION BY SEASON) AS GamesPlayed,
   SUM(FGA) OVER (PARTITION BY SEASON) AS FGA ,SUM(FG3A) OVER (PARTITION BY SEASON) AS FG3A, 
   SUM(FGA-FG3A) OVER (PARTITION BY SEASON) AS FG2A, SUM(FTA) OVER (PARTITION BY SEASON) AS FTA, CEILING (AVG(PTS) OVER (PARTITION BY SEASON)) AS AVGPPG, SUM(AST) OVER (PARTITION BY SEASON) AS SeasonAST
FROM `my-data-project-36654.NBA_Data.NBA_game_detail` AS gd 
JOIN `my-data-project-36654.NBA_Data.NBA_game_dataset` AS g USING (game_id)
WHERE PLAYER_NAME = 'LeBron James'
ORDER BY SEASON)

SELECT 
   SEASON, PLAYER_NAME,TEAM_ABBREVIATION, GAMESPLAYED, FGA, CEILING(FGA/GAMESPLAYED) AS FGAPG, 
   FG3A, CEILING (FG3A/FGA * 100) AS PercentageShotsFG3A  ,CEILING(FG3A/GAMESPLAYED) AS FG3APG, FG2A, CEILING (FG2A/FGA * 100) AS PercentageShotsFG2A,CEILING(FG2A/GAMESPLAYED) AS FG2APG, 
   FTA, CEILING (FTA/GAMESPLAYED) AS FTAPG, CEILING (SeasonAST/GAMESPLAYED) AS APG,AVGPPG
FROM Trends;
~~~~

4. A deeper dive into LeBron James' offesnive game
~~~~ SQL

/* Offensive Breakdown*/
 SELECT 
   DISTINCT SEASON, PLAYER_NAME,TEAM_ABBREVIATION,COUNT (GAME_ID) OVER (PARTITION BY SEASON) AS GameCount, 
   SUM(FG3M) OVER (PARTITION BY SEASON) AS Season_3PTM, SUM(FG3M) OVER (ORDER BY SEASON ASC) AS Run_Total_3PTM,
   SUM(FG3M) OVER (ORDER BY SEASON ASC) * 3 AS PTS_Off_FG3_RUN,
   CEILING(AVG(FG3_PCT) OVER (PARTITION BY SEASON) * 100) AS AVG_FG3_PCT,
   SUM(FGM-FG3M) OVER (PARTITION BY SEASON) AS Season_FG2M, SUM(FGM-FG3M) OVER (ORDER BY SEASON ASC) AS Run_Total_FG2M,
   SUM(FGM-FG3M) OVER (ORDER BY SEASON ASC) * 2 AS PTS_Off_FG2M_RUN,CEILING(AVG ((FGM-FG3M)/ (FGA - FG3A)) OVER (PARTITION BY SEASON) * 100) AS FG2_PCT,
   SUM(FTM) OVER (PARTITION BY SEASON) AS Season_FTM, SUM(FTM) OVER (ORDER BY SEASON ASC) AS Run_Total_FTM,
   SUM(FTM) OVER (ORDER BY SEASON ASC)  AS PTS_Off_FTM_RUN, 
   CEILING(AVG(FT_PCT) OVER (PARTITION BY SEASON) * 100) AS AVG_FT_PCT
 FROM `my-data-project-36654.NBA_Data.NBA_game_detail` AS gd 
 JOIN `my-data-project-36654.NBA_Data.NBA_game_dataset` AS g USING (game_id)
 WHERE PLAYER_NAME = 'LeBron James'
 ORDER BY SEASON ASC;
~~~~

5. LeBron's best statistical season of his career order by descending order in  average points, rebounds, assists, and field goal percentage

~~~~ SQL
/* Best Statistical Season of LBJ Career*/
SELECT 
   DISTINCT SEASON, PLAYER_NAME,TEAM_ABBREVIATION,COUNT (GAME_ID) OVER (PARTITION BY SEASON) AS GameCount,
   ROUND(AVG (PTS) OVER (PARTITION BY season),0) AS AVG_PPG,
   SUM (PTS) OVER (PARTITION BY SEASON) AS Season_PTS_Total, SUM(PTS) OVER (ORDER BY SEASON ASC) AS PTS_Run_Total,
   SUM (PTS) OVER () AS Career_Total_PTS, /* Playoffs and Reg Season*/ 
    ROUND(AVG (REB) OVER (PARTITION BY season),0) AS AVG_RPG, SUM(REB) OVER (PARTITION BY SEASON) AS REB_Season_Total,
    SUM(REB) OVER (ORDER BY SEASON ASC) AS REB_Run_Total,SUM(REB) OVER () Career_REB_Total,
    ROUND(AVG (AST) OVER (PARTITION BY season),0) AS AVG_APG, SUM (AST) OVER (PARTITION BY SEASON) AS AST_Season_Total,
    SUM (AST) OVER (ORDER BY SEASON ASC) AS AST_Run_Total, SUM (AST) OVER () AS Career_AST_Total,
    ROUND(AVG (FG_PCT) OVER (PARTITION BY season),2) * 100.0 AS FG_PCT,  
 FROM `my-data-project-36654.NBA_Data.NBA_game_detail` AS gd 
 JOIN `my-data-project-36654.NBA_Data.NBA_game_dataset` AS g USING (game_id)
 WHERE PLAYER_NAME = 'LeBron James' 
 ORDER BY AVG_PPG DESC, AVG_RPG DESC, AVG_APG DESC, FG_PCT DESC
 LIMIT 1;-- Best statistical season of LBJ career
~~~~

6. LeBron James' career stats
~~~~ SQL 

/* Number of Games Lebron has played per season and ppg and other relevant stats*/
SELECT 
   DISTINCT SEASON, PLAYER_NAME,TEAM_ABBREVIATION,COUNT (GAME_ID) OVER (PARTITION BY SEASON) AS GameCount,
   ROUND(AVG (PTS) OVER (PARTITION BY season),0) AS AVG_PPG,
   SUM (PTS) OVER (PARTITION BY SEASON) AS Season_PTS_Total, SUM(PTS) OVER (ORDER BY SEASON ASC) AS PTS_Run_Total,
   SUM (PTS) OVER () AS Career_Total_PTS, /* Playoffs and Reg Season*/ 
    ROUND(AVG (REB) OVER (PARTITION BY season),0) AS AVG_RPG, SUM(REB) OVER (PARTITION BY SEASON) AS REB_Season_Total,
    SUM(REB) OVER (ORDER BY SEASON ASC) AS REB_Run_Total,SUM(REB) OVER () Career_REB_Total,
    ROUND(AVG (AST) OVER (PARTITION BY season),0) AS AVG_APG, SUM (AST) OVER (PARTITION BY SEASON) AS AST_Season_Total,
    SUM (AST) OVER (ORDER BY SEASON ASC) AS AST_Run_Total, SUM (AST) OVER () AS Career_AST_Total,
    ROUND(AVG (FG_PCT) OVER (PARTITION BY season),2) * 100.0 AS FG_PCT,  
 FROM `my-data-project-36654.NBA_Data.NBA_game_detail` AS gd 
 JOIN `my-data-project-36654.NBA_Data.NBA_game_dataset` AS g USING (game_id)
 WHERE PLAYER_NAME = 'LeBron James' 
 ORDER BY SEASON;
~~~~
