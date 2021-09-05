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

## Classify Images of Clouds in the Cloud with AutoML Vision

- **AutoML Vision** helps developers with limited ML expertise train high quality image recognition models via an easy to use REST API.

- In this lab you will upload images to Cloud Storage and use them to train a custom model to recognize different three types of clouds(cirrus, cumulus, and cumulonimbus)

- Set up AutoML Vision

  -  **Navigation menu** > **APIs & Services** > **Library** > `Cloud AutoML` > **Enable**

- Cloud Shell

  ```sh
  export PROJECT_ID=$DEVSHELL_PROJECT_ID
  export QWIKLABS_USERNAME=$(whoami)@qwiklabs.net
  export QWIKLABS_USERNAME=$(echo $QWIKLABS_USERNAME | sed -e "s/_/-/g")
  
  ## Add automl permission
  gcloud projects add-iam-policy-binding $PROJECT_ID \
      --member="user:$QWIKLABS_USERNAME" \
      --role="roles/automl.admin"
      
  ## Create a bucket
  gsutil mb -p $PROJECT_ID \
      -c standard    \
      -l us-central1 \
      gs://$PROJECT_ID-vcm/
  ```

- Upload training images to Cloud Storage

  ```sh
  export BUCKET=$PROJECT_ID-vcm
  
  ## copy training data
  gsutil -m cp -r gs://spls/gsp223/images/* gs://${BUCKET}
  ```

- Create a dataset

  - You'll need a CSV file where each row contains a URL to a training image and the associated label for that image.

  ```sh
  ## downlaod csv
  gsutil cp gs://spls/gsp223/data.csv .
  
  ## change bucket name
  sed -i -e "s/placeholder/${BUCKET}/g" ./data.csv
  head -5 data.csv
  # gs://qwiklabs-gcp-03-xxxxxxxxxxxx-vcm/cirrus/1.jpg,cirrus
  # gs://qwiklabs-gcp-03-xxxxxxxxxxxx-vcm/cirrus/10.jpg,cirrus
  # gs://qwiklabs-gcp-03-xxxxxxxxxxxx-vcm/cirrus/11.jpg,cirrus
  # gs://qwiklabs-gcp-03-xxxxxxxxxxxx-vcm/cirrus/12.jpg,cirrus
  # gs://qwiklabs-gcp-03-xxxxxxxxxxxx-vcm/cirrus/13.jpg,cirrus
  
  ## upload to cloud storage bucket
  gsutil cp ./data.csv gs://${BUCKET}
  ```

  -  **Navigation menu** > **Vision** > **Datasets** >  **+ NEW DATASET** > `clouds` > `Single-Label Classification` 
  -  **Select a CSV file on Cloud Storage** > `gs://${BUCKET}/data.csv` > **Continue**

- Inspect images
  - **Images** tab 
  - Review the training images
  - Try filtering by different **labels** in the left menu to images.
  - If any images are labeled incorrectly you can click on the image to switch the label.
  - To see a summary of how many images you have for each label, click on **LABEL STATS** at the top of the page. (Train:Validation:Test = 8:1:1)

- Train your model

  - **Train** tab > **Start Training** > **Continue** > Set **8** node hours > **Start Training**
  - Wait half an hour...
  - Kill time with this [video](https://youtu.be/_2eG8xpRYZ4)

- Evaluate your model

  - Confidence threshold

  - Precision: 83.33%

  - Recall: 83.33%

  - Confusion matrix: This table shows how often the model classified each label correctly and which labels were most often confused for that label.

    | **N/A**          | **cumulus** | **cumulonimbus** | **cirrus** |
    | ---------------- | ----------- | ---------------- | ---------- |
    | **cumulus**      | 1           | 1                | 0          |
    | **cumulonimbus** | 0           | 2                | 0          |
    | **cirrus**       | 0           | 0                | 2          |

- Deploy your model

  -  **Test & Use** tab >  **Deploy model** > **Deploy**
  - Wait 20 minutes.

- Generate predictions

  - Download the [image-1](https://cdn.qwiklabs.com/N2psyplM3kFK9NEjjpak3CPIhh8IurY7Tn9vqzi4r8M%3D), [image-2](https://cdn.qwiklabs.com/GZlBRmAKGzsDoT8yNRCh6VmxflxLEQEkiKPohYwja94%3D)
  - Upload and view the predictions.
  - image1: cirrus/0.9994734
  - image2: cumulonimbus/0.9710835

## Google Assistant: Build a Restaurant Locator with the Places API

- In this lab you built a robust Google Assistant application with Dialogflow and Cloud Function.It takes in a user's current location and restaurant preferences to generate the ideal restaurant for them to visit, complete with names, addresses, and photos.

- Create an Actions project
  -  [Actions on Google Developer Console](http://console.actions.google.com/) > **New Project** > Choose the project > **Import project** > **Actions  Console** >  Click the Project

- Build an Action

  - **Build your action > Add Action(s) > Get Started** > **Custom Intent > BUILD** > Yes & Acceopt term of service >  **Create** (At dialogflow essentials page)
  - **Fulfillment** tab> Enable Webhook > `https://google.com` > **Save**
  
- Build the Default Welcome Intent
  
  - **Intentst** tab > **Default Welcome Intent** > **Responses** section > Delete all of the default > **ADD RESPONSES** > **Text response** `Hello there and welcome to Restaurant Locator! What is your location? ` > **Save**
    - [More example Ref.]([Design Walkthrough](https://developers.google.com/actions/design/walkthrough#write_dialogs))
  
- Build the Custom Intent

  - Click **+** next to the intents tab
  - Name: `get_restaurant`
  - **Add Training Phrases**
    - `345 Spear Street` 
    - `1600 Amphitheatre Parkway, Mountain View`
    - `20 W 34th St, New York, NY 10001`
    - Add `@sys.location` Entity to above three phrases
  - **Action and parameters**
    - Check the **REQUIRED** checkbox for the `@sys.location` entity. This tells Dialogflow not to trigger the intent until the parameter is properly provided by the user.
    - Click **Define prompts** for location (right-hand side) and provide a re-prompt phrase. `What is your address?`
    - Click **+ New Parameter**
      - **Required** - Select the checkbox
      - **Parameter name** - proximity
      - **Entity** - @sys.unit-length
      - **Value** - $proximity
      - **Prompts** - How far are you willing to travel?
    - Click **+ New Parameter**
      - **Required** - Select the checkbox
      - **Parameter name** - cuisine
      - **Entity** - @sys.any
      - **Value** - $cuisine
      - **Prompts** - What type of food or cuisine are you looking for?
  - **Responses** 
    - **+** > **Google Assistant**  > Toggle **Set this intent as end of conversation**
  - **Fulfillment**
    - **Enable fulfillment** > **Enable webhook call for this intent** 
  - **SAVE**

- Enable APIs and retrieve an API key

  - **Navigation menu** >**APIs & Services** > **Library** > `Places API` > Enable
  - **Navigation menu** >**APIs & Services** > **Library** > `Maps JavaScript API` > Enable
  - **Navigation menu** >**APIs & Services** > **Library** > `Geocoding API` > Enable
  - **APIs & Services** > **Credentials** >  **+ CREATE CREDENTIALS** > **API Key**
  - Copy the API key and save it .(AIzaSyC_O4B3A4hKD11AY6BaFhEStJsCfwpz63E)

- Initialize a Cloud Function

  - **Navigation menu** > **Cloud Functions** >  **Create function**

    - Function name: `restaurant_locator`
    - Check **Allow unauthenticated invocations**
    - **Save** > **Next**

  - Entry point: `get_restaurant`

  - index.js

    - Replace `<YOUR_API_KEY_HERE>` (18)
    - Replace `<YOUR_REGION>` to `tw`(26)

    ```js
    'use strict';
    
    const {
      dialogflow,
      Image,
      Suggestions
    } = require('actions-on-google');
    
    const functions = require('firebase-functions');
    const app = dialogflow({debug: true});
    
    function getMeters(i) {
         return i*1609.344;
    }
    
    app.intent('get_restaurant', (conv, {location, proximity, cuisine}) => {
          const axios = require('axios');
          var api_key = "<YOUR_API_KEY_HERE>";
          var user_location = JSON.stringify(location["street-address"]);
          var user_proximity;
          if (proximity.unit == "mi") {
            user_proximity = JSON.stringify(getMeters(proximity.amount));
          } else {
            user_proximity = JSON.stringify(proximity.amount * 1000);
          }
          var geo_code = "https://maps.googleapis.com/maps/api/geocode/json?address=" + encodeURIComponent(user_location) + "&region=<YOUR_REGION>&key=" + api_key;
          return axios.get(geo_code)
            .then(response => {
              var places_information = response.data.results[0].geometry.location;
              var place_latitude = JSON.stringify(places_information.lat);
              var place_longitude = JSON.stringify(places_information.lng);
              var coordinates = [place_latitude, place_longitude];
              return coordinates;
          }).then(coordinates => {
            var lat = coordinates[0];
            var long = coordinates[1];
            var place_search = "https://maps.googleapis.com/maps/api/place/findplacefromtext/json?input=" + encodeURIComponent(cuisine) +"&inputtype=textquery&fields=photos,formatted_address,name,opening_hours,rating&locationbias=circle:" + user_proximity + "@" + lat + "," + long + "&key=" + api_key;
            return axios.get(place_search)
            .then(response => {
                var photo_reference = response.data.candidates[0].photos[0].photo_reference;
                var address = JSON.stringify(response.data.candidates[0].formatted_address);
                var name = JSON.stringify(response.data.candidates[0].name);
                var photo_request = 'https://maps.googleapis.com/maps/api/place/photo?maxwidth=400&photoreference=' + photo_reference + '&key=' + api_key;
                conv.ask(`Fetching your request...`);
                conv.ask(new Image({
                    url: photo_request,
                    alt: 'Restaurant photo',
                  }))
                conv.close(`Okay, the restaurant name is ` + name + ` and the address is ` + address + `. The following photo uploaded from a Google Places user might whet your appetite!`);
            })
        })
    });
    
    exports.get_restaurant = functions.https.onRequest(app);
    ```

  - package.json

    ```json
    {
      "name": "get_reviews",
      "description": "Get restaurant reviews.",
      "version": "0.0.1",
      "author": "Google Inc.",
      "engines": {
        "node": "8"
      },
      "dependencies": {
        "actions-on-google": "^2.0.0",
        "firebase-admin": "^4.2.1",
        "firebase-functions": "1.0.0",
        "axios": "0.16.2"
      }
    }
    ```

  - Deploy

  - After creation completes, click `restaurant_locator`  > **trigger** > Copy **Trigger URL**

- Configure the webhook

  - **Dialogflow console**  > **Fulfillment**
    - URL:  Paste the **Trigger URL**
    - **Save**

- Change your Google permission settings

  - Open [Activity Controls page](https://myaccount.google.com/activitycontrols)
  - Turn on **Web & App Activity**

- Test the application with the Actions simulator

  - **Dialogflow console** > **Integrations** > **PREVIEW MIGRATION** >  **Test**
  - The Actions simulator allows you to test your applications without any hardware.
  - `Talk to my test app`
  - `345 Spear Street San Francisco`
  - `0.5 miles away`
  - `Italian food`

## App Engine: Qwik Start - Python

- The App Engine standard environment is based on container instances running on Google's infrastructure, and allows developers to focus on doing what they do best, writing code.

- Features

  - Persistent storage with queries, sorting, and transactions.
  - Automatic scaling and load balancing.
  - Asynchronous task queues for performing work outside the scope of a request.
  - Scheduled tasks for triggering events at specified times or regular intervals.
  - Integration with other Google cloud services and APIs.

- Enable Google App Engine Admin API

  - **Navigation menu** > **APIs & Services** > **Library** > `App Engine Admin API` > Enable

- Download the Hello World app

  ```sh
  gsutil -m cp -r gs://spls/gsp067/python-docs-samples .
  cd python-docs-samples/appengine/standard_python3/hello_world
  ```

- Test the application

  - `dev_appserver.py app.yaml`(run in cloud shell)

  - **Web preview** > **Preview on port 8080**

- Make a change

  ```sh
  cd python-docs-samples/appengine/standard_python3/hello_world
  vi main.py
  ## Change "Hello World!" to "Hello, World!!!!!"
  ```

  - **Web Preview** > **Preview on port 8080**

- Deploy your app

  ```sh
  gcloud app deploy
  # [1] asia-east2 -> Y
  ```

- View your application

  ```sh
  gcloud app browse
  ## Click on the link
  ```
