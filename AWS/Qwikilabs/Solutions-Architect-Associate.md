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

