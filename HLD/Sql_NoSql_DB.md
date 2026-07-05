# **Sql And NoSql DB**

SQL Database

Stores data in tables (rows & columns).

Example:

Users Table
Id	Name	Age
1	Ankit	25
2	Rahul	28

Relationships are supported.

Users

↓

Orders

↓

Payments

Examples
MySQL
PostgreSQL
SQL Server
Oracle

Characteristics
Fixed Schema
ACID Transactions
Joins
Strong Consistency
Relational Data
Nore optimized indexing, primary keys, foreign keys

NoSQL Database:

Doesn't require data to be stored in tables.

Example (Document):

{
  "id":1,
  "name":"Ankit",
  "orders":[
      {
         "product":"Laptop"
      }
  ]
}

Data is flexible.

Examples
MongoDB
Cassandra
DynamoDB
Couchbase

Characteristics
Flexible Schema
Easy Horizontal Scaling
High Availability
Handles Massive Data
Often Eventually Consistent (depends on the database)

SQL vs NoSQL
SQL	                                        NoSQL
Tables	                                    Documents/Key-Value/Column/Graph
Fixed Schema	                            Flexible Schema
ACID	                                    Often BASE/Eventual Consistency (varies by DB)
Vertical scaling initially	                Horizontal scaling is common
Joins supported	                            Usually avoids joins
Complex queries	                            Simpler queries
Strong consistency	                        Often eventual consistency
Hold Structured Data                        Hold Structured as well as unstructured data


## Real Example
Banking Application
Account

↓

Transfer Money

↓

Balance Update

Needs:

100% accuracy
Transactions
Rollback

Use:

✅ SQL

Why?

If:

A sends ₹1000 to B

Either:

Both balances update

OR

Neither updates

Never half.

Social Media

Suppose:

User

↓

Posts

↓

Likes

↓

Comments

Millions of users.

Schema changes often.

Need massive scalability.

Use:

✅ NoSQL

Example

Instagram Post

{
   "user":"Ankit",
   "caption":"Hello",
   "likes":1500,
   "comments":[]
}

Tomorrow you add:

"location":"Delhi"

No schema migration required.
When to Use SQL

## Choose SQL when:

Banking
Payments
Inventory
Airline Booking
Financial Systems
ERP
Payroll

Reason:

Need transactions and strong consistency.

## When to Use NoSQL

Choose NoSQL when:

Social Media
Chat Applications
Product Catalog
IoT
Gaming
Analytics
Logging

Reason:

Need scalability and flexible data models.

Interview Question
"Which database would you choose for an e-commerce website?"

Answer:

Use both.

Users

↓

SQL

------------------

Orders

↓

SQL

------------------

Products

↓

NoSQL

------------------

Reviews

↓

NoSQL

------------------

Logs

↓

NoSQL

Why?

Orders require transactions.

Product details and reviews are more flexible and scale well with NoSQL.

Interview Summary ⭐
Requirement	    Database
Transactions	SQL
Banking	        SQL
Payments	    SQL
Complex Joins	SQL
Flexible Schema	NoSQL
Massive Scale	NoSQL
High Write Throughput	NoSQL
Social Media	NoSQL

