# [Google Cloud Solutions II: Data and Machine Learning](./Google-Cloud-Solutions-Data-and-Machine-Learning)

## Exploring NCAA Data with BigQuery

- **BigQuery** is Google's fully managed, NoOps, low cost analytics database. With BigQuery you can query terabytes and terabytes of data without managing infrastructure or needing a database administrator

- Find the NCAA public dataset in BigQuery

  - **Navigation menu** > **BigQuery** > **SQL workspace**
  - **+ ADD DATA** > **Explore public datasets** > Type "ncaa basketball" > **Enter**
  - Click on the NCAA Basketball tile, then **View Dataset**.
  - Choose `bigquery-public-data/ncaa_basketball/mbb_pbp_sr` in left pannel
  - **Preview** > **Details**
  - To see how big is the table

- Writing queries

  1. Click **+ Compose New Query**.
  2. Write query.
  3. Run.
  4. See Result

- Example

  - **What types of basketball play events are there?**

  ```sql
  #standardSQL
  SELECT
    event_type,
    COUNT(*) AS event_count
  FROM `bigquery-public-data.ncaa_basketball.mbb_pbp_sr`
  GROUP BY 1
  ORDER BY event_count DESC;
  ```

  - **Which 5 games featured the most three point shots made? How accurate were all the attempts?**

  ```sql
  #standardSQL
  #most three points made
  SELECT
    scheduled_date,
    name,
    market,
    alias,
    three_points_att,
    three_points_made,
    three_points_pct,
    opp_name,
    opp_market,
    opp_alias,
    opp_three_points_att,
    opp_three_points_made,
    opp_three_points_pct,
    (three_points_made + opp_three_points_made) AS total_threes
  FROM `bigquery-public-data.ncaa_basketball.mbb_teams_games_sr`
  WHERE season > 2010
  ORDER BY total_threes DESC
  LIMIT 5;
  ```

  - **Which 5 basketball venues have the highest seating capacity?**

  ```sql
  #standardSQL
  SELECT
    venue_name, venue_capacity, venue_city, venue_state
  FROM `bigquery-public-data.ncaa_basketball.mbb_teams_games_sr`
  GROUP BY 1,2,3,4
  ORDER BY venue_capacity DESC
  LIMIT 5;
  ```

  | 1    | AT&T Stadium                  | 80000 | Arlington    | TX   |
  | ---- | ----------------------------- | ----- | ------------ | ---- |
  | 2    | University of Phoenix Stadium | 72220 | Glendale     | AZ   |
  | 3    | NRG Stadium                   | 71054 | Houston      | TX   |
  | 4    | Georgia Dome                  | 71000 | Atlanta      | GA   |
  | 5    | Lucas Oil Stadium             | 70000 | Indianapolis | IN   |

  - **Which teams played in the highest scoring game since 2010?**

  ```sql
  #standardSQL
  #highest scoring game of all time
  SELECT
    scheduled_date,
    name,
    market,
    alias,
    points_game AS team_points,
    opp_name,
    opp_market,
    opp_alias,
    opp_points_game AS opposing_team_points,
    points_game + opp_points_game AS point_total
  FROM `bigquery-public-data.ncaa_basketball.mbb_teams_games_sr`
  WHERE season > 2010
  ORDER BY point_total DESC
  LIMIT 5;
  ```

  | Row  | scheduled_date | name     | market             | alias | team_points | opp_name  | opp_market         | opp_alias | opposing_team_points | point_total |
  | :--- | :------------- | :------- | :----------------- | :---- | :---------- | :-------- | :----------------- | :-------- | :------------------- | :---------- |
  | 1    | 2017-02-10     | Terriers | Wofford            | WOF   | 131         | Bulldogs  | Samford            | SAM       | 127                  | 258         |
  | 2    | 2017-02-10     | Bulldogs | Samford            | SAM   | 127         | Terriers  | Wofford            | WOF       | 131                  | 258         |
  | 3    | 2017-02-04     | Vikings  | Portland State     | PRST  | 124         | Eagles    | Eastern Washington | EWU       | 130                  | 254         |
  | 4    | 2017-02-04     | Eagles   | Eastern Washington | EWU   | 130         | Vikings   | Portland State     | PRST      | 124                  | 254         |
  | 5    | 2013-11-14     | Pioneers | Sacred Heart       | SHU   | 118         | Crusaders | Holy Cross         | HC        | 122                  | 240         |

  - **Since 2015, what was the biggest difference in final score for a National Championship?**

  ```sql
  #standardSQL
  #biggest point difference in a championship game
  SELECT
    scheduled_date,
    name,
    market,
    alias,
    points_game AS team_points,
    opp_name,
    opp_market,
    opp_alias,
    opp_points_game AS opposing_team_points,
    ABS(points_game - opp_points_game) AS point_difference
  FROM `bigquery-public-data.ncaa_basketball.mbb_teams_games_sr`
  WHERE season > 2015 AND tournament_type = 'National Championship'
  ORDER BY point_difference DESC
  LIMIT 5;
  ```

  | Row  | scheduled_date | name     | market             | alias | team_points | opp_name  | opp_market         | opp_alias | opposing_team_points | point_total |
  | :--- | :------------- | :------- | :----------------- | :---- | :---------- | :-------- | :----------------- | :-------- | :------------------- | :---------- |
  | 1    | 2017-02-10     | Terriers | Wofford            | WOF   | 131         | Bulldogs  | Samford            | SAM       | 127                  | 258         |
  | 2    | 2017-02-10     | Bulldogs | Samford            | SAM   | 127         | Terriers  | Wofford            | WOF       | 131                  | 258         |
  | 3    | 2017-02-04     | Vikings  | Portland State     | PRST  | 124         | Eagles    | Eastern Washington | EWU       | 130                  | 254         |
  | 4    | 2017-02-04     | Eagles   | Eastern Washington | EWU   | 130         | Vikings   | Portland State     | PRST      | 124                  | 254         |
  | 5    | 2013-11-14     | Pioneers | Sacred Heart       | SHU   | 118         | Crusaders | Holy Cross         | HC        | 122                  | 240         |
