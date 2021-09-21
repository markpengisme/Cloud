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

## Introduction to Amazon API Gateway

- Amazon API Gateway is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale.

- In this lab, you will create a simple FAQ microservice. The microservice will return a JSON object containing a random question and answer pair using an **Amazon API Gateway** endpoint that invokes a **Amazon Lambda** function.

  ![arch](https://s3.us-west-2.amazonaws.com/us-west-2-aws-training/awsu-spl/spl-58/2.0.20.prod/scripts/micro-service.png)

- Microservice: The idea of a microservices architecture is to take a large, complex system and break it down into independent, decoupled services that are east to manage and extend.

- **Start Lab**>  **Open Console**

- Create a Lambda Function

  - **Services** > **Lambda** > **Create function**

  - **Author from scratch**

    - Function Name: `FAQ`
    - Runtime: `Node.js 12.x`
    - Execution role: Use an existing-role
    - Existing role: lambda-basic-execution
    - **Create Function**

  - **Code tab** > Paste the code below into `index.js`.(Define FAQ & Return a random FAQ)

    ```js
    var json = {
      "service": "lambda",
      "reference": "https://aws.amazon.com/lambda/faqs/",
      "questions": [{
        "q": "What is AWS Lambda?",
        "a": "AWS Lambda lets you run code without provisioning or managing servers. You pay only for the compute time you consume - there is no charge when your code is not running. With Lambda, you can run code for virtually any type of application or backend service - all with zero administration. Just upload your code and Lambda takes care of everything required to run and scale your code with high availability. You can set up your code to automatically trigger from other AWS services or call it directly from any web or mobile app."
      },{
       "q":"What events can trigger an AWS Lambda function?",
       "a":"You can use AWS Lambda to respond to table updates in Amazon DynamoDB, modifications to objects in Amazon S3 buckets, logs arriving in Amazon CloudWatch logs, incoming emails to Amazon Simple Email Service, notifications sent from Amazon SNS, messages arriving in an Amazon Kinesis stream, client data synchronization events in Amazon Cognito, and custom events from mobile applications, web applications, or other web services. You can also invoke a Lambda function on a defined schedule using the AWS Lambda console."
      },{
       "q":"When should I use AWS Lambda versus Amazon EC2?",
       "a":"Amazon Web Services offers a set of compute services to meet a range of needs. Amazon EC2 offers flexibility, with a wide range of instance types and the option to customize the operating system, network and security settings, and the entire software stack, allowing you to easily move existing applications to the cloud. With Amazon EC2 you are responsible for provisioning capacity, monitoring fleet health and performance, and designing for fault tolerance and scalability. AWS Elastic Beanstalk offers an easy-to-use service for deploying and scaling web applications in which you retain ownership and full control over the underlying EC2 instances. Amazon Elastic Container Service is a scalable management service that supports Docker containers and allows you to easily run distributed applications on a managed cluster of Amazon EC2 instances. AWS Lambda makes it easy to execute code in response to events, such as changes to Amazon S3 buckets, updates to an Amazon DynamoDB table, or custom events generated by your applications or devices. With Lambda you do not have to provision your own instances; Lambda performs all the operational and administrative activities on your behalf, including capacity provisioning, monitoring fleet health, applying security patches to the underlying compute resources, deploying your code, running a web service front end, and monitoring and logging your code. AWS Lambda provides easy scaling and high availability to your code without additional effort on your part."
      },{
        "q":"What kind of code can run on AWS Lambda?",
        "a":"AWS Lambda offers an easy way to accomplish many activities in the cloud. For example, you can use AWS Lambda to build mobile back-ends that retrieve and transform data from Amazon DynamoDB, handlers that compress or transform objects as they are uploaded to Amazon S3, auditing and reporting of API calls made to any Amazon Web Service, and server-less processing of streaming data using Amazon Kinesis."
      },{
        "q":"What languages does AWS Lambda support?",
        "a":"AWS Lambda supports code written in Node.js (JavaScript), Python, and Java (Java 8 compatible). Your code can include existing libraries, even native ones. Lambda functions can easily launch processes using languages supported by Amazon Linux, including Bash, Go, and Ruby. Please read our documentation on using Node.js, Python and Java."
      },{
        "q":"Can I access the infrastructure that AWS Lambda runs on?",
        "a":"No. AWS Lambda operates the compute infrastructure on your behalf, allowing it to perform health checks, apply security patches, and do other routine maintenance."
      },{
        "q":"How does AWS Lambda isolate my code?",
        "a":"Each AWS Lambda function runs in its own isolated environment, with its own resources and file system view. AWS Lambda uses the same techniques as Amazon EC2 to provide security and separation at the infrastructure and execution levels."
      },{
        "q":"How does AWS Lambda secure my code?",
        "a":"AWS Lambda stores code in Amazon S3 and encrypts it at rest. AWS Lambda performs additional integrity checks while your code is in use."
      },{
        "q":"What is an AWS Lambda function?",
        "a":"The code you run on AWS Lambda is uploaded as a Lambda function. Each function has associated configuration information, such as its name, description, entry point, and resource requirements. The code must be written in a stateless style i.e. it should assume there is no affinity to the underlying compute infrastructure. Local file system access, child processes, and similar artifacts may not extend beyond the lifetime of the request, and any persistent state should be stored in Amazon S3, Amazon DynamoDB, or another Internet-available storage service. Lambda functions can include libraries, even native ones."
      },{
        "q":"Will AWS Lambda reuse function instances?",
        "a":"To improve performance, AWS Lambda may choose to retain an instance of your function and reuse it to serve a subsequent request, rather than creating a new copy. Your code should not assume that this will always happen."
      },{
        "q":"What if I need scratch space on disk for my AWS Lambda function?",
        "a":"Each Lambda function receives 500MB of non-persistent disk space in its own /tmp directory."
      },{
        "q":"Why must AWS Lambda functions be stateless?",
        "a":"Keeping functions stateless enables AWS Lambda to rapidly launch as many copies of the function as needed to scale to the rate of incoming events. While AWS Lambda's programming model is stateless, your code can access stateful data by calling other web services, such as Amazon S3 or Amazon DynamoDB."
      },{
        "q":"Can I use threads and processes in my AWS Lambda function code?",
        "a":"Yes. AWS Lambda allows you to use normal language and operating system features, such as creating additional threads and processes. Resources allocated to the Lambda function, including memory, execution time, disk, and network use, must be shared among all the threads/processes it uses. You can launch processes using any language supported by Amazon Linux."
      },{
        "q":"What restrictions apply to AWS Lambda function code?",
        "a":"Lambda attempts to impose few restrictions on normal language and operating system activities, but there are a few activities that are disabled: Inbound network connections are managed by AWS Lambda, only TCP/IP sockets are supported, and ptrace (debugging) system calls are restricted. TCP port 25 traffic is also restricted as an anti-spam measure."
      },{
        "q":"How do I create an AWS Lambda function using the Lambda console?",
        "a":"You can author the code for your function using the inline editor in the AWS Lambda console. You can also package the code (and any dependent libraries) as a ZIP and upload it using the AWS Lambda console from your local environment or specify an Amazon S3 location where the ZIP file is located. Uploads must be no larger than 50MB (compressed). You can use the AWS Eclipse plugin to author and deploy Lambda functions in Java and Node.js. If you are using Node.js, you can author the code for your function using the inline editor in the AWS Lambda console. Go to the console to get started."
      },{
        "q":"How do I create an AWS Lambda function using the Lambda CLI?",
        "a":"You can package the code (and any dependent libraries) as a ZIP and upload it using the AWS CLI from your local environment, or specify an Amazon S3 location where the ZIP file is located. Uploads must be no larger than 50MB (compressed). Visit the Lambda Getting Started guide to get started."
      },{
        "q":"Which versions of Python are supported?",
        "a":"Lambda provides a Python 2.7-compatible runtime to execute your Lambda functions. Lambda will include the latest AWS SDK for Python (boto3) by default."
      },{
        "q":"How do I compile my AWS Lambda function Java code?",
        "a":"You can use standard tools like Maven or Gradle to compile your Lambda function. Your build process should mimic the same build process you would use to compile any Java code that depends on the AWS SDK. Run your Java compiler tool on your source files and include the AWS SDK 1.9 or later with transitive dependencies on your classpath. For more details, see our documentation."
      },{
        "q":"What is the JVM environment Lambda uses for execution of my function?",
        "a":"Lambda provides the Amazon Linux build of openjdk 1.8."
      }
      ]
    }
    
    exports.handler = function(event, context) {
        var rand = Math.floor(Math.random() * json.questions.length);
        console.log("Quote selected: ", rand);
    
        var response = {
            body: JSON.stringify(json.questions[rand])
        };
        console.log(response);
        context.succeed(response);
    };
    ```

  - **Deploy**

  - **Configuration** tab > **General configuration** > **Edit**

    - Description: `Provide a random FAQ`
    - **Save**

  - **Add trigger**

    - Select a trigger: API Gateway
    - API: Create an API
    - API type: Rest API
    - Security: Open
    - Expand Additional settings
    - API name: `FAQ-API`
    - Deployment stage: `myDeployment`
    - **Add**
  
- Test tha Lmbda Function

  - Click **API Gateway** in at Function View > **Details**
  - Click **API endpoint** link > You will see a random FAQ entry
  - **Test** tab > **New Event**
    - Name: `BasicTest`
    - Json: `{}`
    - **Save changes**
    - **Test**
  - Execution result: succeeded  > **Details**
  - **Monitor** tab > **View logs in CloudWatch**
  - Click on one of the log streams to see log events.

- **Sign Out** > **End Lab**> **OK**
