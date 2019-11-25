# check-apache-drill-mysql
Check MySQL usability via apache drill

### Prerequisites

#### Prepare data

1. Ensure that docker compose is up (`docker-compose up -d`)
1. Connect via any MySQL IDE as:
    * Host = 127.0.0.1
    * Port = _3305_ for mysql_5 or _3308_ for mysql_8
    * User = root
    * Password = test_pass
1. In mysql_8 Db run:
    ```sql
    CREATE SCHEMA db8;
    USE db8;
    CREATE TABLE company
    (
        id int auto_increment
            primary key,
        name varchar(30) null
    );
    INSERT INTO company (name) 
           VALUES ('c1'), ('c2'), ('c3'), ('c4'), ('c5');
    ```
1. In mysql_5 Db run:
    ```sql
    CREATE SCHEMA db5;
    USE db5;
    CREATE TABLE user
    (
        id int auto_increment
            primary key,
        name varchar(40) null,
        company_id int null
    );
    INSERT INTO user (name, company_id)
           VALUES ('u1', 1), ('u2', 2), ('u3', 6), ('u4', 7), ('u5', 5);
    ```

#### Install Apache Drill:

Follow the official documentation in order to install Apache Drill on 
[Linux or Mac](https://drill.apache.org/docs/installing-drill-on-linux-and-mac-os-x/)
or [Windows](https://drill.apache.org/docs/installing-drill-on-windows/)

#### Setup Apache Drill DB connections:

1. Ensure that docker compose is up (`docker-compose up -d`)
1. Download MySQL 8 JDBC driver from https://dev.mysql.com/downloads/connector/j/
    1. Choose "Platform Independent"
    1. Download zip or tar
    1. Unarchive it to `<path ot apache drill folder>jars/3rdparty directory`
1. Start embedded drill from terminal: `<path ot apache drill folder>/bin/drill-embedded` 
1. Open http://127.0.0.1:8047/storage in web browser
1. Click "Create" under Plugin Management
1. Set mysql_8 connection:
    1. Name = `test_mysql_8`
    1. Configuration:
    ```json
    {
      "type": "jdbc",
      "driver": "com.mysql.jdbc.Driver",
      "url": "jdbc:mysql://localhost:3308",
      "username": "root",
      "password": "test_pass",
      "enabled": true
    }  
    ```
1. Set mysql_5 connection:
    1. Name = `test_mysql_5`
    1. Configuration:
    ```json
    {
      "type": "jdbc",
      "driver": "com.mysql.jdbc.Driver",
      "url": "jdbc:mysql://localhost:3305",
      "username": "root",
      "password": "test_pass",
      "enabled": true
    }  
    ```    

### Testing:

1. Ensure that docker compose is up (`docker-compose up -d`)
1. Ensure that apache drill is up (`<path ot apache drill folder>/bin/drill-embedded`)
2. In terminal drill embedded session run:
```sql
    SELECT u.id   AS user_id
         , u.name AS user_name
         , c.id   AS company_id
         , c.name AS company_name 
      FROM test_mysql_5.db5.user    AS u 
INNER JOIN test_mysql_8.db8.company AS c 
        ON c.id = u.company_id;
```
1. Observe that data from 2 separate servers is joined.
1. Stop apache drill `!exit`
1. Stop docker compose `docker-compose down -v`
