class: middle, center

# intro-db
![db-image](./images/postgresql-logo.png)

---

# Agenda

1. Introduction with example
2. Types of databases
    - Relational and non relational
    - SQL
3. Workshop

---

# An Example

![dvc-schedule](./images/dvc-schedule.png)

---

# An Example

- Problem: storing information about 70,000 lines worth of DVC courses in a text file is inefficient, hard to manage, and slow at insertion, updating, and retrieval

- Parsing the file, then storing the information in data structures such as hash tables is not great, becuase all that information is stored in RAM

- Facebook stores 300 PB of data, an impossible amount to store in RAM

- Updating information requires writing to the file

- Many applications may want to access the data at one time

---

# Introduction

- Understanding databases is a big plus when finding internships - many ask for some knowledge of SQL
- Database Management Systems (DBMS) - Software that allows access to a database
- "Database" may refer to the both the data and the DBMS
- DBMSs structure data according to different models
- DBMSs use different querying languages to access data
    - SQL is the most popular, and is used with many DBMSs
    - Others use their own query languages

![dbms](./images/dbms.png)

---

# Relational Databases

- Use tables of columns and rows to represent data
    - Relate tables together using keys
    - Most use SQL (Structured Query Language)
    - Not very hip
    - Examples: Oracle, MySQL, SQLite

![relational-example](./images/relational-example.png) ![table-example](./images/table-example.png)

---

# NoSQL Databases

- A broad term referring to databases that do not use the relational table model

- Key-Value Stores
    - Act similarly to a hash table
    - Records can hold different types of data
    - Examples: Redis, Amazon DynamoDB

![key-value-example](./images/key-value-example.png)

---

# NoSQL Databases
- Document Stores 
    - Similar to key-value stores - items may have different structures
    - can have a nested structure, like JSON
    - useful for holding arrays
    - Examples: MongoDB, RethinkDB

![document-store-example](./images/document-store-example.png)
# Database design

Think about what data actually needs to be stored. Store it in a way that minimizes data redundancy. Think about what data types you will need and how many objects you have. Each object usually gets its own table.

**Primary keys**

Each table has a Primary Key (PK) column that can be used to uniquely identify a specific row. Also called IDs. They should be created whenever a new row is made and never changed. These are used when referring to data from other tables. Can be as simple as an auto-incrementing number.
    
---

# Database design - Relationships

- One-to-many, aka arrays. Example: a teacher has many classes, but a class only has one teacher. Can't create a column of type array, so need to create a new table.

    ![one-to-many](./images/one-to-many.png)

---

# Database design - Relationships

- Example of arrays with one-to-many

    ![array-example](./images/array-example.png)

---

# Database design - Relationships

- Many-to-many: a customer's order may contain one or more products; and a product can appear in many orders

    ![many-to-many](./images/many-to-many.png)

---

# Database design - Relationships

- One-to-one: Product may contain optional data. We only create a row in `productDetails` if the optional data is chosen.

    ![one-to-one](./images/one-to-one.png)

---

# Workshop Setup

1. Sign up at [heroku.com](https://www.heroku.com)
2. Grab the heroku toolbelt at [toolbelt.heroku.com](https://toolbelt.heroku.com)
    - If on Mac, `brew install heroku-toolbelt`

```bash
$ heroku login
Enter your Heroku credentials.
Email: jesse@example.com
Password (typing will be hidden):
Authentication successful.

$ mkdir intro-db
$ cd intro-db
$ git init
Initialized empty Git repository in ./intro-db/.git/

$ heroku create
Creating app... done, stack is cedar-14
https://***.herokuapp.com/ | https://git.heroku.com/***.git
```

---

# Workshop Setup

- Create the database

    ```bash
    $ heroku addons:create heroku-postgresql:hobby-dev
    Creating postgresql-trapezoidal-55564... done, (free)
    Adding postgresql-trapezoidal-55564 to ...... done
    Setting DATABASE_URL and restarting ...... done, v3
    Database has been created and is available
     ! This database is empty. If upgrading, you can transfer
     ! data from another database with pg:copy
    Use `heroku addons:docs heroku-postgresql` to view documentation.
    ```

- Download data

    ```bash
    $ heroku run bash
    Running bash on ....... up, run.7217

    ~ $ wget https://github.com/dvcoders/intro-db/releases/download/1.0.0/StateNames.csv.tar.gz
    ...
    Saving to: 'StateNames.csv.tar.gz'
    100%[=================================>] 31,686,560  41.7MB/s   in 0.7s
    2016-04-07 20:37:54 (41.7 MB/s) - 'StateNames.csv.tar.gz' saved [31686560/31686560]

    ~ $ tar -xf StateNames.csv.tar.gz
    ```

---

# Workshop Setup

- Connect to the database

    ```bash
    ~ $ psql $DATABASE_URL
    Running psql $DATABASE_URL on ....... up, run.1083
    ...
    d2bh5qbs8qns39=> 
    ```

- Create the table

    ```sql
    => CREATE TABLE names (
        id int,
        name varchar(50),
        year int,
        gender varchar(1),
        state varchar(20),
        count int
    );
    ```
    
- Import data

    ```sql
    => \copy names FROM './StateNames.csv' WITH CSV;
    
    COPY 5647426
    ```

---

# Workshop

Structure of database

```sql
=> \d
            List of relations
 Schema | Name  | Type  |     Owner
--------+-------+-------+----------------
 public | names | table | cjbawbbpgwxjuc
(1 row)

=> \d names
            Table "public.names"
 Column |         Type          | Modifiers
--------+-----------------------+-----------
 id     | integer               |
 name   | character varying(50) |
 year   | integer               |
 gender | character varying(1)  |
 state  | character varying(20) |
 count  | integer               |
```

---

# Workshop

Sample queries

```sql
=> SELECT * FROM names WHERE name='Jesse' and state='CA' and year=1996;
   id   | name  | year | gender | state | count
--------+-------+------+--------+-------+-------
 491828 | Jesse | 1996 | F      | CA    |    28
 658580 | Jesse | 1996 | M      | CA    |  1191
(2 rows)

=> SELECT name, state FROM names WHERE year=1988 ORDER BY count DESC LIMIT 5;
    name     | state
-------------+-------
 Michael     | CA
 Jessica     | CA
 Christopher | CA
 Michael     | NY
 Daniel      | CA
(5 rows)

=> SELECT SUM(count) FROM names WHERE year BETWEEN 1999 AND 2001;
   sum
---------
 9780944
(1 row)
```

---

# Workshop

More sample queries

```sql
=> SELECT DISTINCT name FROM names ORDER BY name ASC LIMIT 5;
  name
---------
 Aaban
 Aadan
 Aadarsh
 Aaden
 Aadhav
(5 rows)

=> SELECT year, SUM(count) FROM names GROUP BY year ORDER BY sum DESC LIMIT 5;
 year |   sum
------+---------
 1957 | 4002231
 1959 | 3955363
 1960 | 3948284
 1958 | 3933693
 1961 | 3929909
(5 rows)
```
