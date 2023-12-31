# DataX DatabendWriter
[简体中文](./databendwriter-CN.md) | [English](./databendwriter.md)

## 1 Introduction
Databend Writer is a plugin for DataX to write data to Databend Table from dataX records.
The plugin is based on [databend JDBC driver](https://github.com/databendcloud/databend-jdbc) which use [RESTful http protocol](https://databend.rs/doc/integrations/api/rest)
to execute query on open source databend and [databend cloud](https://app.databend.com/).

During each write batch, databend writer will upload batch data into internal S3 stage and execute corresponding insert SQL to upload data into databend table.

For best user experience, if you are using databend community distribution, you should try to adopt [S3](https://aws.amazon.com/s3/)/[minio](https://min.io/)/[OSS](https://www.alibabacloud.com/product/object-storage-service) as its underlying storage layer since 
they support presign upload operation otherwise you may expend unneeded cost on data transfer.

You could see more details on the [doc](https://databend.rs/doc/deploy/deploying-databend)

## 2 Detailed Implementation
Databend Writer would use DataX to fetch records generated by DataX Reader, and then batch insert records to the designated columns for your databend table.

## 3 Features
### 3.1 Example Configurations
* the following configuration would read some generated data in memory and upload data into databend table

#### Preparation
```sql
--- create table in databend
drop table if exists datax.sample1;
drop database if exists datax;
create database if not exists datax;
create table if not exsits datax.sample1(a string, b int64, c date, d timestamp, e bool, f string, g variant);
```

#### Configurations
```json
{
  "job": {
    "content": [
      {
        "reader": {
          "name": "streamreader",
          "parameter": {
            "column" : [
              {
                "value": "DataX",
                "type": "string"
              },
              {
                "value": 19880808,
                "type": "long"
              },
              {
                "value": "1926-08-08 08:08:08",
                "type": "date"
              },
              {
                "value": "1988-08-08 08:08:08",
                "type": "date"
              },
              {
                "value": true,
                "type": "bool"
              },
              {
                "value": "test",
                "type": "bytes"
              },
              {
                "value": "{\"type\": \"variant\", \"value\": \"test\"}",
                "type": "string"
              }

            ],
            "sliceRecordCount": 10000
          }
        },
        "writer": {
          "name": "databendwriter",
          "parameter": {
            "username": "databend",
            "password": "databend",
            "column": ["a", "b", "c", "d", "e", "f", "g"],
            "batchSize": 1000,
            "preSql": [
            ],
            "postSql": [
            ],
            "connection": [
              {
                "jdbcUrl": "jdbc:databend://localhost:8000/datax",
                "table": [
                  "sample1"
                ]
              }
            ]
          }
        }
      }
    ],
    "setting": {
      "speed": {
        "channel": 1
       }
    }
  }
}
```

### 3.2 Configuration Description
* jdbcUrl
  * Description: JDBC Data source url in Databend. Please take a look at repository for detailed [doc](https://github.com/databendcloud/databend-jdbc)
  * Required: yes
  * Default: none
  * Example: jdbc:databend://localhost:8000/datax
* username
  * Description: Databend user name
  * Required: yes
  * Default: none
  * Example: databend
* password
  * Description: Databend user password
  * Required: yes
  * Default: none
  * Example: databend
* table
  * Description: A list of table names that should contain all of the columns in the column parameter.
  * Required: yes
  * Default: none
  * Example: ["sample1"]
* column
  * Description: A list of column field names that should be inserted into the table. if you want to insert all column fields use `["*"]` instead.
  * Required: yes
  * Default: none
  * Example: ["a", "b", "c", "d", "e", "f", "g"]
* batchSize
  * Description: The number of records to be inserted in each batch.
  * Required: no
  * Default: 1024
* preSql
  * Description: A list of SQL statements that will be executed before the write operation.
  * Required: no
  * Default: none
* postSql
  * Description: A list of SQL statements that will be executed after the write operation.
  * Required: no
  * Default: none
* writeMode
  * Description：The write mode, support `insert` and `replace` two mode.
  * Required：no
  * Default：insert
  * Example："replace"
* onConflictColumn
  * Description：On conflict fields list.
  * Required：no
  * Default：none
  * Example：["id","user"]

### 3.3 Type Convert
Data types in datax can be converted to the corresponding data types in databend. The following table shows the correspondence between the two types.

| DataX Type | Databend Type                                             |
|------------|-----------------------------------------------------------|
| INT        | TINYINT, INT8, SMALLINT, INT16, INT, INT32, BIGINT, INT64 |
| LONG       | TINYINT, INT8, SMALLINT, INT16, INT, INT32, BIGINT, INT64 |
| STRING     | STRING, VARCHAR                                           |
| DOUBLE     | FLOAT, DOUBLE                                             |
| BOOL       | BOOLEAN, BOOL                                             |
| DATE       | DATE, TIMESTAMP                                           |
| BYTES      | STRING, VARCHAR                                           |


## 4 Performance Test


## 5 Restrictions
Currently, complex data type support is not stable, if you want to use complex data type such as tuple, array, please check further release version of databend and jdbc driver.

## FAQ
