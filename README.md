# Trino(Presto) Server Quickly Build

This project contains a docker image for trino(presto) server which also integrates 
trino client.

## Start up a Trino(Presto) Server
Go to the compose directory and run

`docker compose -p presto up`

Then, log in the Trino(Presto) container and run

`presto-cli --server localhost:8080 --catalog hive`
to use trino client to create a connect with trino server.

Then run

`show schemas;`

If everything is ok, you will get 
```$xslt
--------------------
 default
 information_schema
(2 rows)
```

## What is Behind the Trino(Presto) Server
This trino(presto) server uses hive connector to get data stored 
in localstack s3 and hive metastore to save metadata.
And if you want to upload custom data in order to use trino to query against them,
there are some tips to help you(need to install aws client, `aws_access_key_id` and 
`aws_secret_access_key` are both `test`).
And part of these tips are from https://janakiev.com/blog/presto-trino-s3/#hive-standalone-metastore.

1: Create bucket with
```
aws --endpoint-url=http://localhost:4566 s3api create-bucket --bucket test-data --create-bucket-configuration LocationConstraint=us-west2
```                                                        

2: Go to the root directory of this project and run
```
aws --endpoint-url=http://localhost:4566 s3api put-object --bucket test-data --key iris_parquet/iris.parq --body iris.parq
```
to upload data to the specified bucket.

3: Use trino client to access trino server and run
```
CREATE SCHEMA IF NOT EXISTS hive.iris
WITH (location = 's3a://test-data/');
```
to create schema.

4: Create a table
```
CREATE TABLE IF NOT EXISTS hive.iris.iris_parquet (
  sepal_length DOUBLE,
  sepal_width  DOUBLE,
  petal_length DOUBLE,
  petal_width  DOUBLE,
  class        VARCHAR
)
WITH (
  external_location = 's3a://test-data/iris_parquet',
  format = 'PARQUET'
);

```
5: Run a query
```
SELECT 
  sepal_length,
  class
FROM hive.iris.iris_parquet 
LIMIT 10;
```
If it goes well, you can get the result from trino server.
```
sepal_length |    class
--------------+-------------
          5.1 | Iris-setosa
          4.9 | Iris-setosa
          4.7 | Iris-setosa
          4.6 | Iris-setosa
          5.0 | Iris-setosa
          5.4 | Iris-setosa
          4.6 | Iris-setosa
          5.0 | Iris-setosa
          4.4 | Iris-setosa
          4.9 | Iris-setosa
(10 rows)
```
