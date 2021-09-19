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
