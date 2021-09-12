# Other Lab

[toc]

## Introduction to Amazon Simple Storage Service (S3)

- This lab teaches you the basic feature functionality of Amazon S3 using the AWS Management Console.

- Amazon Simple Storage Service (Amazon S3) is an object storage service that offers industry-leading scalability, data availability, security, and performance. 

- **Start Lab** > **Open Console**

- Create a bucket

  - **Services** > **S3** > **Create bucket** 
    - Bucket name: `reportbucket-xxxxxx`
    - **Create bucket**

- Upload an object to the bucket

  - Download this [jpg](https://s3.us-west-2.amazonaws.com/us-west-2-aws-training/awsu-spl/SPL-TF-100-STESS3-2/2.4.4.prod/images/new-report.png)
  - **reportbucket-xxxxxx** > **Upload** > **Add Files** > `new-report.png `> **Upload **> **Close**

- Make an object public

  - Select **new-report.png** > **Copy URL** > Open new tab and navigate to the URL > Show ''AccessDenied'' message
  - Select **new-report.png** > **Actions** > **Make public **> **Make public** >Show " **Failed to edit public access**" > **Close**
  - **Permissions** > **Block public access (bucket settings)** > **Edit** > Deselect the **Block \*all\* public access** option > **Save changes** > `confirm` > **Confirm**
  - Try to Make public again > **Successfully edited public access** > **Close** 
  - Refresh the browser tab that displayed "AccessDenied".(Now it is  publicly accessible)

- Test connectivity from the EC2 instance

  - **Services** > **EC2** > **Instances (running)** > Select **Bastion Host** > **Connect** > **Session Manager** > **Connect**

  - AWS Systems Manager (SSM) Window

    ```sh
    cd ~
    pwd
    aws s3 ls
    aws s3 ls s3://reportbucket-xxxxxx
    cd reports
    ls
    ## copy file from ec2 to s3
    aws s3 cp report-test1.txt s3://reportbucket-xxxxxx
    ## upload failed: ./report-test1.txt to s3://reportbucket-xxxxxx/report-test1.txt An error occurred (AccessDenied) when calling the PutObject operation: Access Denied
    ```

- Create a bucket policy

  - Download this [file](https://s3.us-west-2.amazonaws.com/us-west-2-aws-training/awsu-spl/SPL-TF-100-STESS3-2/2.4.4.prod/images/v1/sample-file.txt)

  - **Services** > **S3** > **reportbucket-xxxxxx** > **Upload** > **Add Files** > `sample-file.txt `> **Upload** > **Close**

  - Select **sample-file.txt** > **Copy URL** > Open new tab and navigate to the URL > Show ''AccessDenied'' message

  - **Services > IAM > Roles** > Search `EC2InstanceProfileRole` and click >  Record the **Role ARN**

    > Amazon Resource Names (ARN)s uniquely identify AWS resources across all of AWS. 
    > `arn:partition:service:region:account-id:resource`
    
  -  Copy the **Role ARN**
  
  - **Services** > **S3** > **reportbucket-xxxxxx** >  **Permissions** tab >  Bucket Policy **Edit** > Record the **Bucket ARN **> **Policy generator**

    - Select Type of Policy:  **S3 Bucket Policy**.
    - Effect: select **Allow**.
    - Principal: Paste the **EC2 Role ARN** 
    - AWS Service: **Amazon S3**.
    - Actions: **PutObject** & **GetObject**
    - Amazon Resource Name (ARN): Paste the **Bucket ARN** and add `/*` at the end
    - **Add Statement**
    - **Generate Policy**
    - Copy Policy JSON and paste it into Bucket **Bucket policy editor**
    - **Save Changes**
  
  - AWS Systems Manager (SSM) Window
  
    ```sh
    cd ~
    pwd
    cd reports
    ls
    aws s3 ls s3://reportbucket-xxxxxx # new-report.png & sample-file.txt
    
    ## copy file from ec2 to s3
    aws s3 cp report-test1.txt s3://reportbucket-xxxxxx # upload ...
    aws s3 ls s3://reportbucket-xxxxxx # 3 files
    
    ## copy file from s3 to ec2
    aws s3 cp s3://reportbucket-xxxxxx/sample-file.txt sample-file.txt # download ...
    ls sample-file.txt # exist
    ```
  
  - Refresh the browser tab that displayed "AccessDenied", it still does not work
  
  - Add a policy in Statement(**Bucket policy editor**)

    ```json
    {
        "Sid": "Stmt1627422672414",
        "Effect": "Allow",
        "Principal": "*",
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::reportbucket-xxxxxx/*"
    }
    ```
  
  - Refresh the browser tab that displayed "AccessDenied", it can read the `sample-file.txt` now.
  
- Explore versioning

  - **Services** > **S3** > **reportbucket-xxxxxx** > **Properties** > **Bucket Versioning** > Select **Enable** > **Save changes**

    > Versioning is enabled for an entire bucket and all objects within the bucket. It cannot be enabled for individual objects.
    
  - Download this [file](https://s3.us-west-2.amazonaws.com/us-west-2-aws-training/awsu-spl/SPL-TF-100-STESS3-2/2.4.4.prod/images/v2/sample-file.txt)
  
  - **Objects** tab > Upload `sample-file.txt`(new version)
  
  - Refresh the `sample-file.txt`  browser tab, It changes contents to the new version.
  
  - Click  **sample-file.txt** > **Versions** tab > Select `*null`  > **Open** > It show the original version.
  
  - However, if you try to access the older version of the `sample-file.txt` file using the object URL link, you will receive an access denied message. 
  
  - In order to access a previous version of the object, you need to update your bucket policy to include the **"s3:GetObjectVersion"** permission.
  
    ```json
    {
        "Sid": "Stmt1627422672414",
        "Effect": "Allow",
        "Principal": "*",
        "Action": [
                  "s3:GetObject",
                  "s3:GetObjectVersion"
        ],
        "Resource": "arn:aws:s3:::reportbucket-xxxxxx/*"
    }
    ```
  
  - Refresh the object URL link page, It works now.
  
  - **Services** > **S3** > **reportbucket-xxxxxx**  > Toggle  **Show versions**  to on 
  
    - Now you can view the available versions of each object and identify which version is the latest.
  
  - Toggle  **Show versions**  to off > Delete the **sample-file.txt** 
  
    - The **sample-file.txt** object is no longer displayed in the bucket.
    - Toggle  **Show versions**  to on, and the sample-file.txt object is displayed again, but the most recent version is a **Delete marker**.
    - Delete **sample-file.txt** object with the **Delete marker** and toggle  **Show versions**  to off.
    - Notice that the sample-file.txt object has been restored to the bucket.
  
  - Now toggle  **Show versions**  to on, and delete the latest versions.
  
    - After that, toggle **Show versions**  to off, then copy URL and paste to the new tab.
    - The text of the original version of the sample-file.txt object is displayed.
  
- **Sign Out **> **End Lab**  > **OK**

- Conclusion:

  - Create a bucket in Amazon S3
  - Add an object to your bucket
  - Manage access permissions on an object
  - Create a bucket policy
  - Use bucket versioning
