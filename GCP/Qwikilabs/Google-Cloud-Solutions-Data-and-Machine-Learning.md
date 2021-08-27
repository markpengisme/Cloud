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

## TensorFlow for Poets

- [TensorFlow](http://tensorflow.org/) is an open source library for numerical computation, specializing in machine learning applications.

- In this lab, you will learn how to install and run TensorFlow 1.x on a single machine, then train a simple classifier to classify images of flowers.

- This lab uses transfer learning to train your machine.The model for this lab is trained on the [ImageNet](http://image-net.org/), [Large Visual Recognition Challenge dataset](http://www.image-net.org/challenges/LSVRC/2012). These models can differentiate between 1,000 different classes, like "Zebra", "Dalmatian", and "Dishwasher"

- Create a development machine in Compute Engine

  - **Navigation menu** > **Compute Engine** > **VM Instances**  > **Create Instance**
  - **Name:** devhost
  - **Region:** us-central1
  - **Zone:** us-central1-a
  - **Series:** N1
  - **Machine Type:** n1-highcpu-4
  - **Boot disk:**  Ubuntu 18.04 LTS
  - **Identity and API access:** Allow full access to all Cloud APIs
  - Create

- Install TensorFlow(SSH instance)

  ```sh
  sudo apt-get update
  
  ## install python, pip, virturalenv
  sudo apt-get install -y python-pip python-dev python3-pip python3-dev virtualenv
  pip install --upgrade pip
  pip install virtualenv
  virtualenv -p python3 venv
  source venv/bin/activate
  
  ## install tensorflow
  pip install --upgrade tensorflow==1.15.2
  python
  ```

  ```python
  import tensorflow as tf
  hello = tf.constant('Hello, TensorFlow!')
  sess = tf.Session()
  print(sess.run(hello))
  exit()
  ```

- (Re)training the network

  - The retrain script can retrain either [Inception V3 model](https://github.com/tensorflow/models/tree/master/research) or a [MobileNet](https://research.googleblog.com/2017/06/mobilenets-open-source-models-for.html).In this lab, you will use a MobileNet.
  - The principal difference is that Inception V3 is optimized for accuracy, while the MobileNets are optimized to be small and efficient, at the cost of some accuracy.

  ```sh
  git clone https://github.com/googlecodelabs/tensorflow-for-poets-2
  cd tensorflow-for-poets-2
  
  ## training images
  curl http://download.tensorflow.org/example_images/flower_photos.tgz | tar xz
  ls flower_photos
  mv flower_photos tf_files
  
  ## MobileNet configuration
  # 128/160/192/224
  export IMAGE_SIZE=224
  # 1/0.75/0.5/0.25
  export ARCHITECTURE="mobilenet_0.50_${IMAGE_SIZE}"
  ```

  - Open a firewall for Tensorboard on port 6006

    - **VPC network** > **Firewall** > **Create Firewall Rule**
    - **Name:** tensorboard
    - **Targets:** All instances in the network
    - **Source IP ranges:** 0.0.0.0/0
    - **Protocol and ports:** tcp 6006

  - Start TensorBoard(SSH)

    ```sh
    tensorboard --logdir tf_files/training_summaries &
    ## if error -> pkill -f "tensorboard" -> redo
    ```

    - Open new tab: <http://[devhost_External_IP]:6006>

- Run the training

  - We're only training the final layer of that network

  ```sh
  ## retraining script (tf1.x provided)
  python -m scripts.retrain -h
  
  ## This script downloads the pre-trained model, adds a new final layer, and trains that layer on the flower photos you've downloaded to the lab.
  python -m scripts.retrain \
    --bottleneck_dir=tf_files/bottlenecks \
    --how_many_training_steps=500 \
    --model_dir=tf_files/models/ \
    --summaries_dir=tf_files/training_summaries/"${ARCHITECTURE}" \
    --output_graph=tf_files/retrained_graph.pb \
    --output_labels=tf_files/retrained_labels.txt \
    --architecture="${ARCHITECTURE}" \
    --image_dir=tf_files/flower_photos
  ```

  - A **bottleneck** is an informal term often used for the layer just before the final output layer that actually does the classification.

  - **Taining accuracy**/**Validation accuracy**/**Cross entropy**

  - The training's objective is to make the cross entropy as small as possible

    > By default, this script runs 4,000 training steps. Each step chooses 10 images at random from the training set, finds their bottlenecks from the cache, and feeds them into the final layer to get predictions. Those predictions are then compared against the actual labels to update the final layer's weights through a backpropagation process.

  - Go back to TensorBoard tab to see what your results look like.

- Using the Retrained Model

  ```sh
  ## daisy
  python -m scripts.label_image \
      --graph=tf_files/retrained_graph.pb  \
      --image=tf_files/flower_photos/daisy/21652746_cc379e0eea_m.jpg
  # daisy (score=0.97501)
  
  ## roses
  python -m scripts.label_image \
      --graph=tf_files/retrained_graph.pb  \
      --image=tf_files/flower_photos/roses/2414954629_3708a1a04d.jpg
  # tulips (score=0.78116)
  ```

- Trying other hyperparameters

  - Use your image on scripts.label_image
  - Use small learning_rate(`--learning_rate=0.5`) on scripts.retrain
  - Use your images (`--image_dir`) on scripts.retrain
    - Each sub-folder is named after one of your categories and contains only images from that category

## Creating an Object Detection Application Using TensorFlow

- This lab will show you how to install and run an object detection application. The application uses **TensorFlow2** and [Object Detection API](https://github.com/tensorflow/models/tree/master/research/object_detection) libraries to detect multiple objects in an uploaded image.

- Pre-trained models: [the COCO dataset](http://mscoco.org/) are capable of detecting general objects in 80 categories.(Higher COCO mAP indicate better accuracy)

  | Pre-trained model name                                       | Speed  | COCO mAP |
  | ------------------------------------------------------------ | ------ | -------- |
  | [ssd_mobilenet_v1_coco](http://download.tensorflow.org/models/object_detection/ssd_mobilenet_v1_coco_11_06_2017.tar.gz) | fast   | 21       |
  | [ssd_inception_v2_coco](http://download.tensorflow.org/models/object_detection/ssd_inception_v2_coco_11_06_2017.tar.gz) | fast   | 24       |
  | [rfcn_resnet101_coco](http://download.tensorflow.org/models/object_detection/rfcn_resnet101_coco_11_06_2017.tar.gz) | medium | 30       |
  | [faster_rcnn_resnet101_coco](http://download.tensorflow.org/models/object_detection/faster_rcnn_resnet101_coco_11_06_2017.tar.gz) | medium | 32       |
  | [faster_rcnn_inception_resnet_v2_atrous_coco](http://download.tensorflow.org/models/object_detection/faster_rcnn_inception_resnet_v2_atrous_coco_11_06_2017.tar.gz) | slow   | 37       |

- Launch a VM instance

  - **Navigation menu** > **Compute Engine** > **VM Instances**  > **Create Instance**
  - **Machine type**: Custom(4,16)
  - **Firewall:** Allow HTTP traffic
  - **Networking**: Click default > **External IP** > **Create IP address** > Name:staticip > **Reserve**
  - Create

- SSH into the instance

  ```sh
  sudo -i
  
  ## Install the python flask tensorflow
  apt-get update
  apt-get install -y protobuf-compiler python3-pil python3-lxml python3-pip python3-dev git
  
  pip3 install --upgrade pip
  pip3 install Flask==1.1.1 WTForms==2.2.1 Flask_WTF==0.14.2 Werkzeug==0.16.0
  pip3 install tensorflow
  
  ## Install the Object Detection API library:
  cd /opt
  git clone https://github.com/tensorflow/models
  cd models/research
  protoc object_detection/protos/*.proto --python_out=.
  
  ## Download the pre-trained model binaries
  mkdir -p /opt/graph_def
  cd /tmp
  for model in \
    ssd_mobilenet_v1_coco_11_06_2017 \
    ssd_inception_v2_coco_11_06_2017 \
    rfcn_resnet101_coco_11_06_2017 \
    faster_rcnn_resnet101_coco_11_06_2017 \
    faster_rcnn_inception_resnet_v2_atrous_coco_11_06_2017
  do \
    curl -OL http://download.tensorflow.org/models/object_detection/$model.tar.gz
    tar -xzf $model.tar.gz $model/frozen_inference_graph.pb
    cp -a $model /opt/graph_def/
  done
  
  ## choose a model(faster_rcnn_resnet101_coco_11_06_2017) for the web application
  ln -sf /opt/graph_def/faster_rcnn_resnet101_coco_11_06_2017/frozen_inference_graph.pb /opt/graph_def/frozen_inference_graph.pb
  ```

- Install and launch the web application

  ```sh
  cd $HOME
  git clone https://github.com/GoogleCloudPlatform/tensorflow-object-detection-example
  ## app
  cp -a tensorflow-object-detection-example/object_detection_app_p3 /opt/
  chmod u+x /opt/object_detection_app_p3/app.py
  ## service
  cp /opt/object_detection_app_p3/object-detection.service /etc/systemd/system/
  systemctl daemon-reload
  systemctl enable object-detection
  systemctl start object-detection
  systemctl status object-detection
  ```

- Test the web application

  - <http://[instance1_External_IP]:6006>
  - Username - `username`
  - Password - `passw0rd`
  - **Choose File** > **Upload** > wait 30 seconds to shows the result
  - The object names detected by the model are shown to the right of the image, in the application window.
  - Click an object name to display rectangles surrounding the corresponding objects in the image.
  - The rectangle thickness increases with object identification confidence.

- Change the inference model

  ```sh
  systemctl stop object-detection
  MODEL_NAME=faster_rcnn_inception_resnet_v2_atrous_coco_11_06_2017
  ln -sf /opt/graph_def/${MODEL_NAME}/frozen_inference_graph.pb /opt/graph_def/frozen_inference_graph.pb
  systemctl start object-detection
  systemctl status object-detection
  ```
