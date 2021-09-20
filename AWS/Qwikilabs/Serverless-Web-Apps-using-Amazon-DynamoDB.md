# Serverless Web Apps using Amazon DynamoDB

[toc]

## Introduction to Amazon DynamoDB

- **Amazon DynamoDB** is a fast and flexible NoSQL database service for all applications that need consistent, low latency at any scale.

- In this lab, you will create a table in Amazon DynamoDB to store information about a music library. Then, query and delete the table.

- **Start Lab**>  **Open Console**

- Create a New Table

  - **AWS Management Console** > **Services** > **DynamoDB** > **Create table**

    - Table name: Music
    - PK: Artist
    - Select **Add sort key**: Song
    - **Create**

    > The Primary Key is used to partition data across DynamoDB servers.
    >
    > The combination of Primary Key and Sort Key uniquely identifies each item in a DynamoDB table.

  - **Items** > **Create item**

    - Artist: `Pink Floyd`
    - Song: `Money`
    - Click **(+)** **> Append** > **String** >`Album` String: `The Dark Side of the Moon`
    - Click **(+)** **> Append** > **Number** >`Year` Number: `1973`
    - **Save**

  - Create a Second Item

    - `Artist`:String `John Lennon`
    - `Song`:String `Imagine`
    - `Album`: String `Imagine`
    - `Year`: Number `1971`
    - `Genre`: String `Soft rock`

  - Create a third Item

    - `Artist`: String `Psy`
    - `Song`: String `Gangnam Style`
    - `Album`: String `Psy 6 (Six Rules), Part 1`
    - `Year`: Number `2011`
    - `LengthSeconds`: Number `219`

  - Above show that you can add attributes to one item that may be different to the attributes on other items.(Flexible)

- Modify an Existing Item

  - Click **Psy** > `2011->2012` > **Save**

- Query the Table

  - There are two ways to query a DynamoDB table
    - Query: Finds items based on PK and optionally Sort Key. It is fully indexed, so it runs very fast.
    - Scan: Look through every item in a table. it is less efficient and can take significant time for larger tables.
  - Change **Scan** to **Query** > Enter `Psy` and `Gangnam Style` > Click **Start search**
  - Change **Query** to **Scan** > **Add filter** >`Year Number = 1971` > Click **Start search**

- Delete the Table
  - Click **Delete table** > Type **Delete** > Click **Delete**
- **Sign Out** > **End Lab**> **OK**

## Introduction to AWS Lambda

- **AWS Lambda** is a compute service that runs your code in response to events and automatically manages the compute resources for you, making it easy to build applications that respond quickly to new information.

- **Start Lab**>  **Open Console**

- Scenario: Serverless image thumbnail application
  ![flow](https://s3.us-west-2.amazonaws.com/us-west-2-aws-training/awsu-spl/spl-88/2.3.15.prod/images/overview1.png)
  1. A user uploads an object to the source bucket in **Amazon S3** (object-created event).
  2. Amazon S3 detects the object-created event.
  3. Amazon S3 publishes the object-created event to AWS Lambda by invoking the Lambda function and passing event data as a function parameter.
  4. AWS Lambda executes the Lambda function.
  5. From the event data it receives, the Lambda function knows the source bucket name and object key name. The Lambda function reads the object and creates a thumbnail using graphics libraries, then saves the thumbnail to the target bucket.
  
- Create the Amazon S3 Buckets
  - **AWS Management Console** > **Services** > **S3**
  - **Create bucket** (Source)
    - Bucket Name: `image-xxxxxx`
    - Click **Create bucket**
  - **Create bucket** (Target)
    - Bucket Name: `image-xxxxxx-resized`
    - Click **Create bucket**
  - Download testing [image]([HappyFace.jpg](https://s3.us-west-2.amazonaws.com/us-west-2-aws-training/awsu-spl/spl-88/2.3.15.prod/images/HappyFace.jpg))
  - Click **image-xxxxxx** > **Upload**  > **Add files** > `HappyFace.jpg` > **Upload**
  
- Create an AWS Lambda Function
  - **Services** > **Lambda** > **Create function** > **Author from scratch**
    - Function name: **Create-Thumbnail**
    - Runtime: **Python 3.7**
    - Execution role: **Use an existing role**
    - Existing role:  **lambda-execution-role**
    - **Create function**
  
- **Add trigger**

  - **S3**
  - Bucket: `image-xxxxxx`
  - Event type: **All object create events**
  - Select **I ack...**
  - **Add**

- **Code** tab >  Click **Upload from** > **Amazon S3 loaction** >

  - `https://s3-us-west-2.amazonaws.com/us-west-2-aws-training/awsu-spl/spl-88/2.3.15.prod/scripts/CreateThumbnail.zip`

  - **Save**

  - Examine the code

     1. Get an event and extract it to bucket and key
     2. Download the images and `resize_image`
     3. Upload thre resized image to the `resized bucket`

    ```python
    import boto3
    import os
    import sys
    import uuid
    from PIL import Image
    import PIL.Image
    
    s3_client = boto3.client('s3')
    
    def resize_image(image_path, resized_path):
        with Image.open(image_path) as image:
            image.thumbnail((128, 128))
            image.save(resized_path)
    
    def handler(event, context):
        for record in event['Records']:
            bucket = record['s3']['bucket']['name']
            key = record['s3']['object']['key']
            download_path = '/tmp/{}{}'.format(uuid.uuid4(), key)
            upload_path = '/tmp/resized-{}'.format(key)
    
            s3_client.download_file(bucket, key, download_path)
            resize_image(download_path, upload_path)
            s3_client.upload_file(upload_path, '{}-resized'.format(bucket), key)
    ```

  - **Runtime settings** > **Edit**

    - Handler: `CreateThumbnail.handler`
    - **Save**

  - **Configuration** tab > **General configuration** > **Edit**

    - Description: `Create a thumbnail-sized image`
    - Memory: use default value
    - Timeout: use default value
    - **Save**

  - **Test Your Function**

    - **Test** tab > **New event**
      - Template: **Amazon S3 Put**
      - Name: **Upload**
      - Replace **example-bucket** in template
        - `"name": "images-xxxxxx",`
        - `"arn": "arn:aws:s3:::images-xxxxxx"`
      - Replace `test/key` to  `HappyFace.jpg`
      - **Test**
    - **Execution result: succeeded** > Click **Details** to dosplay information
    - **Services** > **S3** > Click **image-xxxxxx-resized** > Select **HappyFace.jpg** > **Actions** > **Open** > The image should now be a smaller thumbnail of the original image.
  
- Monitoring and Logging

  - **Services** > **Lambda** > Click **Create-Thumbnail**  > **Monitor** tab > Graphs show these metrics
    1. Invocations
    2. Duration
    3. Error count and success rate (%)
    4. Throttles
    5. Iterator Age
    6. Concurrent executions
  - Click **View logs in CloudWatch**
    - Click the **Log Stream** that appears
    - Expand each message to view the log message details.
  - **Sign Out** > **End Lab**> **OK**
