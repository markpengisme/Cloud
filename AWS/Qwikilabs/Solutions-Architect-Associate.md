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

## Creating an Amazon Virtual Private Cloud (VPC) with AWS CloudFormation

- This lab will demonstrate how to create an Amazon Virtual Private Cloud (VPC) network using AWS CloudFormation.

- You will then launch the AWS CloudFormation template to create a four-subnet Amazon VPC that spans two Availability Zones and a NAT that allows servers in the private subnets to communicate with the Internet in order to download packages and updates.

- **AWS CloudFormation** gives developers and systems administrators an easy way to create and manage a collection of related AWS resources, provisioning and updating them in an orderly and predictable fashion.

- **Start lab** > **Open Console**

- Deploy a Stack using AWS CloudFormation

  - Download [file](https://s3.us-west-2.amazonaws.com/us-west-2-aws-training/awsu-spl/spl-15/4.1.9.prod/scripts/vpc-1.yaml)

  - **AWS Management Console** > **Services** > **CloudFormation** > **Create stack** > **With new resources**

    1. **Specify template**: **Upload a template file** >  **Choose file** > `vpc-1.yaml` > **Next**

    2. **Specify stack detail**: Stack Name: `Lab` > **Next**

    3. **Configure stack options**: **Next**

    4. **Review Lab**: **Next**

  - **Stack info** > Wait for Status:**CREATE_COMPLETE**

  - **Resources** > Display the resources

- Examine the VPC

  - ![image-20210726063128243](/Users/pengshengmao/Library/Application Support/typora-user-images/image-20210726063128243.png)

  - **Services** > **VPC** > **Filter by VPC** > Select **Lab VPC**> **Your VPCs** > Select **Lab VPC**

    - A VPC is an isolated section of the AWS Cloud that allows resources to communicate with each other and, selectively, with the Internet. When deploying resources such as Amazon EC2 instances, you must select the VPC in which the instance will be launched.

    - **Details** > **IPv4 CIDR** = 10.0.0.0/16(Have all 10.0.x.x IP)

    ```yaml
    ## vpc-1.yaml
    Resources:
      VPC:
        Type: AWS::EC2::VPC
        Properties:
          CidrBlock: 10.0.0.0/16
          EnableDnsHostnames: true
          Tags:
          - Key: Name
            Value: Lab VPC
    ```

  - **VPC Management Console** > **Internet Gateways** >

    - An **Internet gateway** is a horizontally scaled, redundant, and highly available VPC component that allows communication between instances in your VPC and the Internet. It, therefore, imposes no availability risks or bandwidth constraints on your network traffic. Purpose
    - An Internet gateway serves two purposes: to provide a target in your VPC route tables for Internet-routable traffic and to perform network address translation (NAT) for instances that have been assigned public IPv4 addresses.

    ```yaml
    ## vpc-1.yaml
    Resources:
      #...
      InternetGateway:
        Type: AWS::EC2::InternetGateway
        Properties:
          Tags:
          - Key: Name
            Value: Lab Internet Gateway
    ```

    - A VPC Gateway Attachment creates a relationship between a VPC and a gateway, such as this Internet Gateway.

    ```yaml
    ## vpc-1.yaml
    Resources:
      #...
      AttachGateway:
        Type: AWS::EC2::VPCGatewayAttachment
        Properties:
          VpcId: !Ref VPC
          InternetGatewayId: !Ref InternetGatewa
    ```

  - **VPC Management Console** > **Subnets**

    - **Public Subnet 1** is connected to the Internet,  using the **Internet Gateway**(Let resources in this subnet be publicly accessible)
    - **Private Subnet 1** is *not* connected to the Internet.(More secure for resources in this subnet)
    - Properties: VpcId/CidrBlock/AvailabilityZone

    ```yaml
    ## vpc-1.yaml
    Resources:
      #...
      PublicSubnet1:
        Type: AWS::EC2::Subnet
        Properties:
          VpcId: !Ref VPC
          CidrBlock: 10.0.0.0/24
          AvailabilityZone: !Select
            - '0'
            - !GetAZs ''
          Tags:
            - Key: Name
              Value: Public Subnet 1
    
      PrivateSubnet1:
        Type: AWS::EC2::Subnet
        Properties:
          VpcId: !Ref VPC
          CidrBlock: 10.0.1.0/24
          AvailabilityZone: !Select
            - '0'
            - !GetAZs ''
          Tags:
            - Key: Name
              Value: Private Subnet 1
    ```

  - **VPC Management Console** > **Route Table** > Select **Public Route Table** > **Routes**

    - For traffic within the VPC (10.0.0.0/16), route the traffic locally.
    - For traffic going to the Internet (0.0.0.0/0), route the traffic to the Internet Gateway.

    ```yaml
    ## vpc-1.yaml
    Resources:
      #...
      PublicRouteTable:
        Type: AWS::EC2::RouteTable
        Properties:
          VpcId: !Ref VPC
          Tags:
            - Key: Name
              Value: Public Route Table
      PublicRoute:
        Type: AWS::EC2::Route
        Properties:
          RouteTableId: !Ref PublicRouteTable
          DestinationCidrBlock: 0.0.0.0/0
          GatewayId: !Ref InternetGateway
    ```

    - **Private Route Table** is similar to Public Route Table, but only Public Route Table have **PublicRoute** which is what makes it public.

    - **Subnet associations** > Show Public Route Table is associated with **Public Subnet 1**

      - A Route Table can be associated with multiple subnets, with each association requiring an explicit linkage, as the following  code:

      ```yaml
      ## vpc-1.yaml
      Resources:
        #...
        PublicSubnetRouteTableAssociation1:
          Type: AWS::EC2::SubnetRouteTableAssociation
          Properties:
            SubnetId: !Ref PublicSubnet1
            RouteTableId: !Ref PublicRouteTable
      ```

  - **Services** > **CloudFormation** > **Lab** > **Outputs**

    - Show the CloudFormation template has been configured to return information about the resources it created:
    - **VPC** is the ID of the VPC that was created.
    - **AZ1** shows the Availability Zone in which the Subnets were created.

    ```yaml
    Outputs:
      VPC:
        Description: VPC
        Value: !Ref VPC
    
      AZ1:
        Description: Availability Zone 1
        Value: !GetAtt
          - PublicSubnet1
          - AvailabilityZone
    ```

- Updating a Stack

  - Update the stack to two public subnets and two private subnets with the template for high availability.

    ![img](https://s3.us-west-2.amazonaws.com/us-west-2-aws-training/awsu-spl/spl-15/4.1.9.prod/images/vpc-2.png)

  - Download the [file](https://s3.us-west-2.amazonaws.com/us-west-2-aws-training/awsu-spl/spl-15/4.1.9.prod/scripts/vpc-2.yaml)

  - Select **Lab** > **Update** > Select **Replace current template** > Select **Upload a template file** > Click **Choose file** > `vpc-2.yaml` > **Next** > **Next** > **Next** > Look at the **Change set preview**, there are 2 subnets and 2 route table associations > **Update stack**

  - **Stack info** > Wait for the status change to**UPDATE_COMPLETE**

  - **Outputs** > Additional AZ2 is added and has a different value to AZ1

  - **Services** > **VPC** > **Filter by VPC** > Select **Lab VPC**> **Subnet** > There are 4 subnets.

- Viewing a Stack in CloudFormation Designer

  - **AWS CloudFormation Designer** is a graphic tool for creating, viewing, and modifying AWS CloudFormation templates.
  - **Services** > **CloudFormation** > Click **Lab** stack > **Template** tab > **View in Designer**
  - Examine the diagram
    - Use the Zoom controls and drag the image.
    - Arrows show the relationship between resources.
    - Click the **Components** tab at the bottom and click on some elements in the diagram. It will show the code that defines the resources.(YAML/JSON)

- Delete the Stack
  - Click the **Close** at the top-left of the page to **Leave Page**.
  - Click the **Lab** stack > **Delete** >  **Delete stack** >  **Events**: wait for all status change to **DELETE_COMPLETE**
  - **Services** > **VPC** > **Your VPCs** > The **Lab VPC** is no longer present.

- **End lab** > **OK**
