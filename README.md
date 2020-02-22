![npm version](https://img.shields.io/node/v/xmysql.svg)
[![Build Status](https://travis-ci.org/o1lab/xmysql.svg?branch=master)](https://travis-ci.org/o1lab/xmysql)
[![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/o1lab/xmysql/master/LICENSE)

# xMySQL : One command to generate REST APIs

# Why this ?

Generating REST APIs for a MySQL database which does not follow conventions

<p align="center">
  <img src="./assets/rick_and_morty.gif" alt="xmysql gif"/>
</p>



# Setup and Usage

xmysql requires node >= 7.6.0

```
npm install -g xmysql
```
```
xmysql -h localhost -u mysqlUsername -p mysqlPassword -d databaseName
```
```
http://localhost:3000
```
<br>

**That's it! Minimalistic**


# Example : Generate REST APIs for any DB
	
<p align="center">
  <img src="./assets/xmysql-runner.gif" alt="xmysql gif" width="750"/>
</p>


# Features
* Generates API for **ANY** MySql database 
* Serves APIs irrespective of naming conventions of primary keys, foreign keys, tables etc
* Support for composite primary keys 
* REST API Usual suspects : CRUD, List, FindOne, Count, Exists, Distinct
* Bulk insert, Bulk delete, Bulk read 
* Relations
* Pagination 
* Sorting
* Row filtering - Where
* Aggregate functions
* Group By, Having (as query params) 
* Group By, Having (as a separate API) 
* Multiple group by in one API 
* Chart API for numeric column 
* Auto Chart API - (a gift for lazy while prototyping) 
* Supports views  
* Prototyping (features available when using local MySql server only)
    * Run dynamic queries
    * Upload single file
    * Upload multiple files
    * Download file


____


## API Overview

| Type | API URL                               | Comments                                               |
|------|---------------------------------------|--------------------------------------------------------- 
| GET       | /                                | Gets all REST APIs                                     |
| GET       | /api/tableName                   | Lists rows of table                                    |
| POST      | /api/tableName                   | Create a new row                                       |
| PUT       | /api/tableName                   | Replaces existing row with new row                     |
| POST 	    | /api/tableName/bulk              | Create multiple rows - send object array in request body|
| GET       | /api/tableName/bulk              | Lists multiple rows - /api/tableName/bulk?_ids=1,2,3   |
| DELETE    | /api/tableName/bulk              | Deletes multiple rows - /api/tableName/bulk?_ids=1,2,3 |
| GET       | /api/tableName/:id               | Retrieves a row by primary key                         |
| PATCH     | /api/tableName/:id               | Updates row element by primary key                     |
| DELETE    | /api/tableName/:id               | Delete a row by primary key                            |
| GET       | /api/tableName/findOne           | Works as list but gets single record matching criteria |
| GET       | /api/tableName/count             | Count number of rows in a table                        |
| GET       | /api/tableName/distinct          | Distinct row(s) in table - /api/tableName/distinct?_fields=col1|
| GET       | /api/tableName/:id/exists        | True or false whether a row exists or not              |
| GET       | [/api/parentTable/:id/childTable| Get list of child table rows with parent table foreign key   |

# Debugging xmysql in docker.

Given you've deployed your xmysql docker container like

``` docker run -d \
--network local_dev \
--name xmysql \
-p 3000:80 \
-e DATABASE_HOST=mysql_host \
-e DATABASE_USER=root \
-e DATABASE_PASSWORD=password \
-e DATABASE_NAME=sys \
markuman/xmysql:0.4.2
``` 

but the response is just

```["http://127.0.0.1:3000/api/tables","http://127.0.0.1:3000/api/xjoin"]```

then obviously the connection to your mysql database failed.

     // attach to the xmysql image
     docker exec -ti xmysql
     
     // install mysql cli client
     apk --update --no-cache add mysql-client
    
     // try to access your mysql database 
     mysql-client -h mysql_host
    
profit from the mysql-client error output and improve the environment variables for mysql

## Nginx Reverse Proxy Config with Docker

This is a config example when you use nginx as reverse proxy

```
events {
   worker_connections 1024;
   
}

http {
    server {
        server_name 127.0.0.1;
        listen 80 ;
        location / {
            rewrite ^/(.*) /$1 break;
            proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://127.0.0.1:3000;
        }
    }
}

```

**Example**

- create a docker network `docker network create local_dev`
- start a mysql server `docker run -d --name mysql -p 3306:3306 --network local_dev -e MYSQL_ROOT_PASSWORD=password mysql`
- start xmysql `docker run -d --network local_dev --name xmyxql -e DATABASE_NAME=sys -e DATABASE_HOST=mysql -p 3000:80 markuman/xmysql:0.4.2`
- start nginx on host system with the config above `sudo nginx -g 'daemon off;' -c /tmp/nginx.conf`
- profit `curl http://127.0.0.1/api/host_summary_by_file_io_type/describe`

When you start your nginx proxy in a docker container too, use as `proxy_pass` the `--name` value of xmysql. E.g. `proxy_pass http://xmysql` (remember, xmysql runs in it's docker container already on port 80).
Tests : setup on local machine


# Tests : setup on local machine

`docker-compose run test`


