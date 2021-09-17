# Solutions Architect - Associate

## Introduction to AWS Identity and Access Management (IAM)

- This lab shows you how to manage access and permissions to your AWS services using AWS Identity and Access Management (**IAM**). Practice the steps to add users to groups, manage passwords, log in with IAM-created users, and see the effects of IAM policies on access to specific services.

- **Start lab** > **Open Console**

- Explore the users and groups
  - **AWS Management Console** > **Services** > **IAM**>**Users**
    - user-1/user-2/user-3
    - **user-1** > **Permissions** > no permissions
    - **Groups** > no groups
    - **Security credentials** > **Console password**:Enabled
  - **AWS Management Console** > **Services** > **IAM**>**User groups**
    - EC2-admin/EC2-Support/S3-Support
    - **EC2-Support** > **Permissions** >
      - **AmazonEC2ReadOnlyAccess** is `AWS managed`
      - **AmazonEC2ReadOnlyAccess** > **JSON**
        - Effect: Allow or Deny
        - Action: What API calls can be made(e.g.`"Action": "ec2:Describe*"`)
        - Resource: Specifies the objects that the statement covers.(e.g.`arn:aws:s3:::example_bucket` )
    - **User groups** > **S3-Support** > **Permissions** > **AmazonS3ReadOnlyAccess** > **JSON**

      ```json
      {
          "Version": "2012-10-17",
          "Statement": [
              {
                  "Effect": "Allow",
                  "Action": [
                      "s3:Get*",
                      "s3:List*"
                  ],
                  "Resource": "*"
              }
          ]
      }
      ```

    - **User groups** > **EC2-Admin** > **Permissions** > **EC2-Admin-Policy**

      - **EC2-Admin-Policy** is `Customer inline`(assigned to one user or group)
      - **EC2-Admin-Policy** > **Permissions** > **AmazonS3ReadOnlyAccess**> **JSON**
      - Cancel
  
- Scenario: enable permissions to support company

  | User   | In Group    | Permissions                               |
  | :----- | :---------- | :---------------------------------------- |
  | user-1 | S3-Support  | Read-Only access to Amazon S3             |
  | user-2 | EC2-Support | Read-Only access to Amazon EC2            |
  | user-3 | EC2-Admin   | View, Start and Stop Amazon EC2 instances |

  - Add user-1 to S3-Support group
    - **User Groups** > **S3-Support** > **Users** > **Add users** > Select **user-1** > **Add Users**
  - Add user-2 to EC2-Support group in the same method
  - Add user-3 to EC2-Admin group in the same method
  - **User Groups** > each group should have 1 user

- Sign-in and test users

  - **AWS Management Console** > **Services** > **IAM** > **Dashboard** >  copy **Sign-in URL** > open in private window
    - IAM user name: `user-1`
    - Password: **AdministratorPassword** in the lab left panel
    - **Services** > **S3** > **xxxx-s3bucket-xxxx** > have permission to view
    - **Services** > **EC2** > Choose correct region in top-right that match the value in the lab left panel > **Instnaces** > no permission to view
  - **user-1** > **Sign Out** > copy **Sign-in URL** > open in private window
    - IAM user name: `user-2`
    - Password: **AdministratorPassword** in the lab left panel
    - **Services** > **EC2** > Choose correct region in top-right that match the value in the lab left panel > **Instnaces** >  have permission to view
    - Select Instance > **Instance state** > **Stop Instance**> **Error Message(Failed to stop the instance)**
    - **Services** > **S3** >  no permission to view
  - **user-2** > **Sign Out** > copy **Sign-in URL** > open in private window
    - IAM user name: `user-3`
    - Password: **AdministratorPassword** in the lab left panel
    - **Services** > **EC2** > Choose correct region in top-right that match the value in the lab left panel > **Instnaces** >  have permission to view
    - Select Instance > **Instance state** > **Stop Instance**> **Successfully stopped**

- End lab

  - Return to AWS Management Console > **Sign Out** > **End lab** > **OK**

## Introduction to Amazon EC2

- This lab provides you with a basic overview of launching, resizing, managing, and monitoring an Amazon EC2 instance. **Amazon Elastic Comput Cloud**(EC2) is a web service that provides resizable compute capacity in the cloud.

- **Start lab** > **Open Console**

- Launch EC2 instance

  - **Services** > **EC2** > **Launch instance**

  - Step 1: Choose an Amazon Machine Image (AMI)

    - Amazon Machine Image: Amazon Linux AMI

  - Step 2: Choose an Instance Type

    - Instance Type: t3.micro

  - Step 3: Configure Instance Details

    - Network: Lab VPC

    - Select Protect against accidental termination

    - Advanced Details > User data(can do automated configuration tasks)

      ```sh
      #!/bin/bash
      yum -y install httpd
      systemctl enable httpd
      systemctl start httpd
      echo '<html><h1>Hello From Your Web Server!</h1></html>' > /var/www/html/index.html
      ```

  - Step 4: Add Storage

    - EC2 stores data on Elastic-Block-Store(EBS) which is the network-attached virtual disk

  - Step 5: Add Tags

    - Help to categorize instances
    - **Add Tag**
      - Key: Name
      - Value: Web Server

  - Step 6: Configure Security Group

    - Create a new security group( as a virtual firewall controls traffic flow)
      - **Security group name:** Web Server security group
      - **Description:** Security group for my web server
    - Remove SSH Rule for security

  - Step 7: Review Instance Launch

    - **Launch** > Choose **Proceed with a key pair** > Select **I ack...** > **Launch Instance** > **View Instance**

    - Check
      - Instance State: running
      - Status Checks: 2/2 checks passed

    - Select **Web Server** > **Details** displays detailed info

- Monitor instance

  - Select **Web Server** > **Monitoring** > Display CloudWatch metrics
  - **Actions** > **Monitor and troubleshooting** > **Get system log**
    - See the  HTTP package that was installed.
    - **Cancel**
  - **Actions** > **Monitor and troubleshooting** > **Get instance screenshot**
    - For troubleshooting
    - **Cancel**

- Update Your Security Group and Access the Web Server

  - Select **Web Server** > **Details** > **Public IPv4 address** > **open address**
  - can't access the web server cause security group is not permitting inbound traffic on port 80
  - **Security Groups** > Click **Web Server security group**'s **Security group ID**
  - **Edit inbound rules** > **Add rule**
    - Type: HTTP
    - Source: Anywhere
    - **Save rules**
  - Return to web server page and refresh the page, then it will show **Hello From Your Web Server!**

- Resize Your Instance: Instance Type and EBS Volume

  - **EC2 Management Console**  > **Instances** > Select **Web Server** > **Instance state** > **Stop instance** >  **Stop**
  - Select **Web Server** > **Actions** > **Instance settings** >**Change instance type**
    - Instance Type: t3.small
    - **Apply**
  - **EC2 Management Console** > **Volumes** > **Actions** > **Modify Volume**
    - Size: 10
    - **Yes** > **Close**
  - **EC2 Management Console**  > **Instances** > Select **Web Server** > **Start instance**

- Explore EC2 Limits

  - **EC2 Management Console** > **Limits**> **All limits** > Select **Running instances**
  - Running On-Demand All Standard is 1280 vCPUs -> Maximum of 640 t3.micro(need 2 vCPUs) can be started

- Test Termination Protection

  - **EC2 Management Console**  > **Instances** > Select **Web Server** > **Instance state** > **Terminate instance** > **Terminate** > Failed to terminate instances
  - **Actions** > **Instance settings** > **Change termination protection** > Deselect **Enable** > **Save** > **Instance state** > **Terminate instance** > **Terminate** > Successfully terminate instance
  
- **End lab** > **OK**

## Caching Static Files with Amazon CloudFront

- Amazon CloudFront is a content delivery web service. It integrates with other Amazon Web Services products to give developers and businesses an easy way to distribute content to end users with low latency, high data transfer speeds, and no minimum usage commitments.

- If you have a file that is updated frequently, you can  use Amazon CloudFront to cache it, and customize your cache expiration time. You can also choose not to cache a file, and Amazon CloudFront will accelerate content distribution using persistent connections and optimized routing from the origin.

- In this lab, you will learn how to start distributing your content with Amazon CloudFront by taking a simple static website in Amazon S3.

- **Start lab** > **Open Console**

- Verify Your Lab Environment

  - **AWS Management Console** > **Services** > **CloudFormation**
  - Check that is one stack and it's status is **Status**
  - Click the stack > **Outputs**
    - An S3 Bucket(Store static website files)
    - A Static Website URL
  - **Services** > **S3** > Click **xxxx-S3Bucket-xxxx** > Click **cf_lab1.html** > **Copy URL** > open new tab for **S3 Object URL**
  - **Permissions** > show not block all public access
    - By default, your Amazon S3 bucket and all of the objects in it are private
    - If you want to allow anyone to access the objects in your Amazon S3 bucket using Amazon CloudFront URLs
    - So that they're accessible only via Amazon CloudFront using Origin Access Identity.

- Create an Amazon CloudFront Web Distribution

  - **Services** > **CloudFront** > **Create a CloudFront distribution**
  - Origin domain: **S3Bucket**
  - **Create Distribution**
  - **Distributions** > Wait for Status **Enabled**
  - Ref. Distribution Settings Options: **Price Class**/**Alternate Domain Names (CNAMEs)**/**Default Root Object**/**Logging**/**Comment**

- Test Your Static Website from an Amazon CloudFront Distribution

  - In local

    - Create file `cf_lab1_test.html` and paste the code below, then replace **Name** to **Domain Name**.

      ```html
      <html>
        <head>
          My CloudFront Test
        </head>
        <body>
          <p>My text content goes here.</p>
          <p><img src="http://NAME/grand_canyon.jpg" alt="my test image" /></p>
        </body>
      </html>
      ```

    - Open the html file in browser > You should see a web page(The embedded image file served from the edge location that CloudFront determined by speed.)

- Test Your Full Site Hosting

  - Open new tab for `Cloud_Front_Distribution_Domain_Name/cf_lab1.html`
  - Open the new tab with image address
  - Notice the html and image is also hosted by CloudFront

- Update Your Site: Cached Content Considerations

  - Change image name to `grand_canyon_v2.jpg` in **cf_lab1_test.html** and open it.
  - Load the `cf_lab1.html` from S3 amd CloudFront.
  - **AWS Management Console** > **Services** > **S3** > Click **xxxx-S3Bucket-xxxx**  >Select **cf_lab1.html** > **Delete** > type  `permanently delete` > Click **Delete objects** >**Close**
  - Refresh the `cf_lab1.html` from S3 amd CloudFront.
    - S3: AccessDenied
    - CloudFront: Same
  - **S3 Management Console** > **cf_lab1-highlands.html** > **Actions** > **Rename object** > `cf-lab1.html` > **Save**
  - Refresh the `cf_lab1.html` from S3 amd CloudFront.
    - S3: New image
    - CloudFront: Same(Old image)
  - This behavior results from the redundancy and caching behavior of Amazon CloudFront.

- Invalidate Your Cache

  - **AWS Management Console** > **Services** > **CloudFront** > Click your distribution > **Invalidations** tab > **Create Invalidation** > **Object Paths**:`/cf_lab1.html` > **Create Invalidate**
  - Wait for **Status** change to **Completed**
  - Refresh the `cf_lab1.html` from S3 amd CloudFront.
    - S3: Same(New image)
    - CloudFront: New image
  - Since the cache was invalidated, refreshing the page caused Amazon CloudFront to pull the updated file from the origin (Amazon S3) and copied the new version to the cache.

