# Google Developer Essentials

## Weather Data in BigQuery

- Introduction

  - Two public datasets in BigQuery: weather data from NOAA and citizen complaints data from New York City.
  - Benefit: **Serverless**, **Ease of use**, **Scale**, **Shareability**

- Explore weather data

  -  **Navigation menu** > **BigQuery **> **ADD DATA** > **Explore public datasets** > `noaa_gsod` > **VIEW DATASET**

  - **bigquery-public-data** > **noaa_gsod** > **gsod2014** > **Preview** tab

  - Query > RUN -- (Weather AVG in 2 places at 20XX/06/12)

    ```SQL
    SELECT
      -- Create a timestamp from the date components.
      stn,
      TIMESTAMP(CONCAT(year,"-",mo,"-",da)) AS timestamp,
      -- Replace numerical null values with actual null
      AVG(IF (temp=9999.9,
          null,
          temp)) AS temperature,
      AVG(IF (wdsp="999.9",
          null,
          CAST(wdsp AS Float64))) AS wind_speed,
      AVG(IF (prcp=99.99,
          0,
          prcp)) AS precipitation
    FROM
      `bigquery-public-data.noaa_gsod.gsod20*`
    WHERE
      CAST(YEAR AS INT64) > 2010
      AND CAST(MO AS INT64) = 6
      AND CAST(DA AS INT64) = 12
      AND (stn="725030" OR  -- La Guardia
        stn="744860")    -- JFK
    GROUP BY
      stn,
      timestamp
    ORDER BY
      timestamp DESC,
      stn ASC
    ```

- Explore New York citizen complaints data

  - **bigquery-public-data** > **new_york** > **311_service_requests** > **Preview**

  - Query > RUN -- What arethe most common complaints?

    ```SQL
    SELECT
      EXTRACT(YEAR
      FROM
        created_date) AS year,
      complaint_type,
      COUNT(1) AS num_complaints
    FROM
      `bigquery-public-data.new_york.311_service_requests`
    GROUP BY
      year,
      complaint_type
    ORDER BY
      num_complaints DESC
    ```

- Saving a new table of weather data

  - Click on the "..."(View actions icon) to the left of the lab project in the explorer > **Create dataset**

    - Dataset ID: demos
    - **Create dataset**

  - **More** > **Query settings** > **Set a destination table for query results**

    - Project name: `qwikilabs-gpc-xxxx`
    - Dataset name: **demos**
    - Table name: `nyc_weather`
    - Results size:  **Allow large results (no size limit)**
    - **SAVE**

  - Query > RUN

    ```SQL
    SELECT
      -- Create a timestamp from the date components.
      timestamp(concat(year,"-",mo,"-",da)) as timestamp,
      -- Replace numerical null values with actual nulls
      AVG(IF (temp=9999.9, null, temp)) AS temperature,
      AVG(IF (visib=999.9, null, visib)) AS visibility,
      AVG(IF (wdsp="999.9", null, CAST(wdsp AS Float64))) AS wind_speed,
      AVG(IF (gust=999.9, null, gust)) AS wind_gust,
      AVG(IF (prcp=99.99, null, prcp)) AS precipitation,
      AVG(IF (sndp=999.9, null, sndp)) AS snow_depth
    FROM
      `bigquery-public-data.noaa_gsod.gsod20*`
    WHERE
      CAST(YEAR AS INT64) > 2008
      AND (stn="725030" OR  -- La Guardia
           stn="744860")    -- JFK
    GROUP BY timestamp
    ```

  - **More** > **Query settings** > **Save query results in a temporary table**

- Find correlation between weather and complaints using  [CORR](https://docs.oracle.com/cd/B19306_01/server.102/b14200/functions028.htm) function

- Query > RUN (Compare the number of complaints and temperature using the CORR function)

  ```SQL
  SELECT
    descriptor,
    sum(complaint_count) as total_complaint_count,
    count(temperature) as data_count,
    ROUND(corr(temperature, avg_count),3) AS corr_count,
    ROUND(corr(temperature, avg_pct_count),3) AS corr_pct
  From (
  SELECT
    avg(pct_count) as avg_pct_count,
    avg(day_count) as avg_count,
    sum(day_count) as complaint_count,
    descriptor,
    temperature
  FROM (
    SELECT
      DATE(timestamp) AS date,
      temperature
    FROM
      demos.nyc_weather) a
    JOIN (
    SELECT x.date, descriptor, day_count, day_count / all_calls_count as pct_count
    FROM
      (SELECT
        DATE(created_date) AS date,
        concat(complaint_type, ": ", descriptor) as descriptor,
        COUNT(*) AS day_count
      FROM
        `bigquery-public-data.new_york.311_service_requests`
      GROUP BY
        date,
        descriptor)x
      JOIN (
        SELECT
          DATE(timestamp) AS date,
          COUNT(*) AS all_calls_count
        FROM `demos.nyc_weather`
        GROUP BY date
      )y
    ON x.date=y.date
  )b
  ON
    a.date = b.date
  GROUP BY
    descriptor,
    temperature
  )
  GROUP BY descriptor
  HAVING
    total_complaint_count > 5000 AND
    ABS(corr_pct) > 0.5 AND
    data_count > 5
  ORDER BY
    ABS(corr_pct) DESC
  ```

  The results indicate that Heating complaints are negatively correlated with temperature (i.e., more heating calls on cold days) and calls about dead trees are positively correlated with temperature (i.e., more calls on hot days).

  | Row  | descriptor                                     | total_complaint_count | data_count | corr_count | corr_pct |
  | :--- | :--------------------------------------------- | :-------------------- | :--------- | :--------- | :------- |
  | 2    | HEATING: HEAT                                  | 871935                | 998        | -0.799     | -0.799   |
  | 12   | Dead/Dying Tree: Planted More Than 2 Years Ago | 19044                 | 559        | 0.683      | 0.683    |

- Query > RUN (Compare the number of complaints and wind speed with the CORR function.)

  ```SQL
  SELECT
    descriptor,
    sum(complaint_count) as total_complaint_count,
    count(wind_speed) as data_count,
    ROUND(corr(wind_speed, avg_count),3) AS corr_count,
    ROUND(corr(wind_speed, avg_pct_count),3) AS corr_pct
  From (
  SELECT
    avg(pct_count) as avg_pct_count,
    avg(day_count) as avg_count,
    sum(day_count) as complaint_count,
    descriptor,
    wind_speed
  FROM (
    SELECT
      DATE(timestamp) AS date,
      wind_speed
    FROM
      demos.nyc_weather) a
    JOIN (
    SELECT x.date, descriptor, day_count, day_count / all_calls_count as pct_count
    FROM
      (SELECT
        DATE(created_date) AS date,
        concat(complaint_type, ": ", descriptor) as descriptor,
        COUNT(*) AS day_count
      FROM
        `bigquery-public-data.new_york.311_service_requests`
      GROUP BY
        date,
        descriptor)x
      JOIN (
        SELECT
          DATE(timestamp) AS date,
          COUNT(*) AS all_calls_count
        FROM `demos.nyc_weather`
        GROUP BY date
      )y
    ON x.date=y.date
  )b
  ON
    a.date = b.date
  GROUP BY
    descriptor,
    wind_speed
  )
  GROUP BY descriptor
  HAVING
    total_complaint_count > 5000 AND
    ABS(corr_pct) > 0.5 AND
    data_count > 5
  ORDER BY
    ABS(corr_pct) DESC
  ```

  Notice that the Corr columns are both negative for noise related complaints

  | Row  | descriptor                                | total_complaint_count | data_count | corr_count | corr_pct |
  | :--- | :---------------------------------------- | :-------------------- | :--------- | :--------- | :------- |
  | 1    | Noise - Street/Sidewalk: Loud Talking     | 109951                | 452        | -0.668     | -0.668   |
  | 2    | Noise - Vehicle: Car/Truck Music          | 78780                 | 449        | -0.644     | -0.644   |
  | 5    | Noise - Street/Sidewalk: Loud Music/Party | 192111                | 452        | -0.552     | -0.552   |
