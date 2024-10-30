#---------Data postprocessing---------------------------------------------------------------------------------------------------------#

# type: ignore

#------------------------------------------------------------------------------------------------------------------------------------#
# Step 1: Convert .txt file to CSV with ANSI encoding
#------------------------------------------------------------------------------------------------------------------------------------#

# Use PowerShell to convert the .txt file to a CSV file with ANSI encoding 
# from the console output during training:


# Episode: 0, Total Reward: 782.0, Epsilon: 1.0
# Replay Buffer Memory:  306
# Moving Average (Training):  782.0
# Episode: 1, Total Reward: 686.0, Epsilon: 0.9913970890487328
# Replay Buffer Memory:  594
# Moving Average (Training):  734.0
# Episode: 2, Total Reward: 625.0, Epsilon: 0.985377738013741
# Replay Buffer Memory:  797
# Moving Average (Training):  697.6666666666666
# Episode: 3, Total Reward: 625.0, Epsilon: 0.9810417250844651
# Replay Buffer Memory:  944
# Moving Average (Training):  679.5
# Episode: 4, Total Reward: 1075.0, Epsilon: 0.9673058958891547
# ...
# Episode: 14000, Total Reward: 2369.0, Epsilon: 0.02
# Replay Buffer Memory:  300000
# Moving Average (Training):  2521.22


# powershell
# Get-Content "C:\Users\sfadm\Desktop\DoubleDQN_11.txt" | ForEach-Object {
#     if ($_ -match 'Episode') {
#         $episode = ($_ -split ",")[0] -replace 'Episode: ', ''
#         $totalReward = ($_ -split ",")[1] -replace 'Total Reward: ', ''
#     }
#     elseif ($_ -match 'Moving Average') {
#         $movingAverage = ($_ -split ":")[1].Trim()
#         "$episode,$totalReward,$movingAverage" | Out-File "C:\Users\sfadm\Desktop\mario_data.csv" -Append -Encoding ASCII
#     }
# }

#------------------------------------------------------------------------------------------------------------------------------------#
# Step 2: Drop and recreate the table in DuckDB (or other)
#------------------------------------------------------------------------------------------------------------------------------------#

# Recreate the table in DuckDB to store the episode data
# This can be adapted to work with any SQL-based database system, like Postgres or SQLite.

# DROP TABLE IF EXISTS mario_data_ddqn_11;
# CREATE TABLE mario_data_ddqn_11 (
#     Episode INTEGER, 
#     Total_Reward DOUBLE, 
#     Moving_Average DOUBLE
# );

# DROP TABLE IF EXISTS mario_data_ddqn_14;
# CREATE TABLE mario_data_ddqn_14 (
#     Episode INTEGER, 
#     Total_Reward DOUBLE, 
#     Moving_Average DOUBLE,
#     Episode_Length INTEGER,
#     Step_Count    INTEGER,
#     Success       INTEGER
# );

#------------------------------------------------------------------------------------------------------------------------------------#
# Step 3: Import the data into the newly created table
#------------------------------------------------------------------------------------------------------------------------------------#

# SQL query to import data from the CSV into the database.
# For example, in DuckDB or another database, you can read and insert from CSV like this:

# INSERT INTO mario_data_ddqn_11
# SELECT * FROM read_csv_auto('C:/Users/sfadm/Desktop/mario_data.csv');

# INSERT INTO mario_data_ddqn_14
# SELECT * FROM read_csv_auto('C:/Users/sfadm/Desktop/training_log_double_14.txt', delim=',');

INSERT INTO mario_data_ddqn_14 
SELECT * FROM read_csv_auto('C:/Users/sfadm/Desktop/training_log_double_14_complete.txt', delim=','); 


# In case you had to abort the training


# WITH moving_avg_cte AS (
#     SELECT 
#         Episode,
#         AVG(Total_Reward) OVER (
#             ORDER BY Episode 
#             ROWS BETWEEN 99 PRECEDING AND CURRENT ROW
#         ) AS Moving_Average_New
#     FROM mario_data_ddqn_14
# )
# UPDATE mario_data_ddqn_14 t
# SET Moving_Average = m.Moving_Average_New
# FROM moving_avg_cte m
# WHERE t.Episode = m.Episode
# AND t.Moving_Average != m.Moving_Average_New;


#------------------------------------------------------------------------------------------------------------------------------------#
# Step 4: Remove duplicates from the table (sanity check)
#------------------------------------------------------------------------------------------------------------------------------------#

# SQL query to remove any duplicate episodes in the table
# The below query can be used in DuckDB or another SQL-based system.

# Note!: query performs a sanity check by removing duplicate episodes, ensuring 
# that only unique records remain in the table. It ensures data integrity 
# and maintains consistency across the dataset.

# DELETE FROM mario_data_ddqn_11
# WHERE Episode IN (
#     SELECT Episode
#     FROM mario_data_ddqn_11
#     GROUP BY Episode
#     HAVING COUNT(*) > 1
# )
# AND rowid NOT IN (
#     SELECT MIN(rowid)
#     FROM mario_data_ddqn_11
#     GROUP BY Episode
# );

DELETE FROM mario_data_ddqn_14 
WHERE Episode IN ( 
    SELECT Episode
    FROM mario_data_ddqn_14
    GROUP BY Episode
    HAVING COUNT(*) > 1
)
AND rowid NOT IN (
    SELECT MIN(rowid)
    FROM mario_data_ddqn_14
    GROUP BY Episode
);  

# Sanity check 
SELECT * FROM mario_data_ddqn_14;   

#------------------------------------------------------------------------------------------------------------------------------------#
# Step 5: Compute success rate per block of 500 episodes
#------------------------------------------------------------------------------------------------------------------------------------#

# SQL query to compute the success rate of episodes per block (500 or 1000 episodes each)
# This example works in DuckDB or any database that supports window functions.
# To successfully clear Level [1,1], a cumulative reward of at least 3000 points is required, while Level [1,4] demands a minimum of 2138 points.
# Also included, average episode length

# WITH Blocked_Episodes AS (
#     SELECT 
#         Episode,
#         Total_Reward,
#         FLOOR(Episode / 500) AS Block
#     FROM 
#         mario_data_ddqn_11
#        
# ),
# Success_Per_Block AS (
#     SELECT 
#         Block,
#         COUNT(CASE WHEN Total_Reward >= 3000 THEN 1 END) AS Successful_Episodes,
#         COUNT(*) AS Total_Episodes,
#         COUNT(CASE WHEN Total_Reward >= 3000 THEN 1 END) * 100.0 / COUNT(*) AS Success_Rate
#     FROM 
#         Blocked_Episodes
#     GROUP BY 
#         Block
# )

# SELECT 
#     Block * 500 AS Block_Start_Episode, 
#     Block * 500 + 499 AS Block_End_Episode,
#     Successful_Episodes, 
#     Total_Episodes, 
#     Success_Rate
# FROM 
#     Success_Per_Block
# ORDER BY 
#     Block;

# WITH Blocked_Episodes AS (
#     SELECT 
#         Episode,
#         FLOOR(Episode / 500) AS Block,
#         Success
#     FROM 
#         mario_data_ddqn_14
# ),
# Success_Per_Block AS (
#     SELECT 
#         Block,
#         SUM(Success) AS Successful_Episodes,
#         COUNT(*) AS Total_Episodes,
#         SUM(Success) * 100.0 / COUNT(*) AS Success_Rate
#     FROM 
#         Blocked_Episodes
#     GROUP BY 
#         Block
# )
# SELECT 
#     Block * 500 AS Block_Start_Episode, 
#     Block * 500 + 499 AS Block_End_Episode,
#     Successful_Episodes, 
#     Total_Episodes, 
#     Success_Rate
# FROM 
#     Success_Per_Block
# ORDER BY 
#  Block;

#------------------------------------------------------------------------------------------------------------------------------------#
# Step 5.5: Average episode length
#------------------------------------------------------------------------------------------------------------------------------------#

# A longer average episode length suggests that the agent is surviving longer in the environment, which often indicates better decision-making

# SELECT 
#     Block * 500 AS Block_Start_Episode, 
#     Block * 500 + 499 AS Block_End_Episode,
#     Successful_Episodes, 
#     Total_Episodes, 
#     Success_Rate
# FROM 
#     Success_Per_Block
# ORDER BY 
#     Block;

# WITH Blocked_Episodes AS (
#     SELECT 
#         Episode,
#         Episode_Length,
#         FLOOR(Episode / 500) AS Block
#     FROM 
#         mario_data_ddqn_14
# )
# SELECT 
#     Block * 500 AS Block_Start_Episode, 
#     Block * 500 + 499 AS Block_End_Episode,
#     AVG(Episode_Length) AS Average_Episode_Length
# FROM 
#     Blocked_Episodes
# GROUP BY 
#     Block
# ORDER BY 
#     Block;

# Or combine both:

WITH Blocked_Episodes AS (
    SELECT 
        Episode,
        Success,
        Episode_Length,
        FLOOR(Episode / 1000) AS Block
    FROM 
        mario_data_ddqn_14
),
Success_Per_Block AS (
    SELECT 
        Block,
        SUM(Success) AS Successful_Episodes,
        COUNT(*) AS Total_Episodes,
        SUM(Success) * 100.0 / COUNT(*) AS Success_Rate,
        AVG(Episode_Length) AS Average_Episode_Length
    FROM 
        Blocked_Episodes
    GROUP BY 
        Block
)
SELECT 
    Block * 1000 AS Block_Start_Episode, 
    Block * 1000 + 999 AS Block_End_Episode,
    Successful_Episodes, 
    Total_Episodes, 
    Success_Rate,
    Average_Episode_Length
FROM 
    Success_Per_Block
ORDER BY 
    Block;      






#------------------------------------------------------------------------------------------------------------------------------------#
# Step 6: Create the table with moving averages, and calculate min/max over the last 100 episodes for the Double DQN network
#------------------------------------------------------------------------------------------------------------------------------------#

# Create the table with moving averages, and calculate min/max over the last 100 episodes

COPY (WITH Moving_Avg_Window AS (
    SELECT 
        Episode, 
        Moving_Average,
        -- Define the window for the last 100 episodes
        MIN(Moving_Average) OVER (ORDER BY Episode ROWS BETWEEN 99 PRECEDING AND CURRENT ROW) AS min_moving_avg_100,
        MAX(Moving_Average) OVER (ORDER BY Episode ROWS BETWEEN 99 PRECEDING AND CURRENT ROW) AS max_moving_avg_100
    --FROM mario_data_ddqn_11
      FROM mario_data_ddqn_14
)
SELECT * FROM Moving_Avg_Window) 
--TO 'C:/Users/sfadm/Desktop/moving_avg_stats.csv' WITH (HEADER, DELIMITER ','); 
  TO 'C:/Users/sfadm/Desktop/moving_avg_stats_double_14.csv' WITH (HEADER, DELIMITER ','); 


#------------------------------------------------------------------------------------------------------------------------------------#
# Additional potential stats algorithm: Detect streaks of high rewards
#------------------------------------------------------------------------------------------------------------------------------------#

# SQL query to detect streaks of episodes where rewards are over a certain threshold (here 3040).
# This query can be run in DuckDB or other SQL-compliant databases.

# WITH episode_streaks AS (
#     SELECT
#         Episode,
#         Total_Reward,
#         CASE 
#             WHEN Total_Reward > 3040 THEN 1 
#             ELSE 0 
#         END AS reward_over_3000,
#         ROW_NUMBER() OVER (ORDER BY Episode) -
#         ROW_NUMBER() OVER (PARTITION BY CASE WHEN Total_Reward > 3040 THEN 1 ELSE 0 END ORDER BY Episode) AS streak_identifier
#     FROM
#         mario_data_ddqn_11
# ),
# streak_summary AS (
#     SELECT
#         MIN(Episode) AS episode_start,
#         MAX(Episode) AS episode_end,
#         COUNT(*) AS streak_length,
#         STRING_AGG(Total_Reward::TEXT, ',') AS total_reward_array
#     FROM
#         episode_streaks
#     WHERE
#         reward_over_3000 = 1
#     GROUP BY
#         streak_identifier
# )
# SELECT 
#     episode_start AS "Episode Start",
#     episode_end AS "Episode End",
#     streak_length AS "Streak Length",
#     total_reward_array AS "Total Reward Array"
# FROM
#     streak_summary
# ORDER BY
#     streak_length DESC;
