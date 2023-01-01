
~~~ SQL

/* JOINING Datasets to return LeBron's stats from both datasets*/

SELECT *
 FROM `my-data-project-36654.NBA_Data.NBA_game_detail` AS gd 
 JOIN `my-data-project-36654.NBA_Data.NBA_game_dataset` AS g USING (game_id)
 WHERE PLAYER_NAME = 'LeBron James';
~~~~
