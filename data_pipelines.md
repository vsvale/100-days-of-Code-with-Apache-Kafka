- we will need a cluster, schema registry and ksqldb (4 streamin units), [mysql](https://github.com/confluentinc/learn-kafka-courses/blob/main/data-pipelines/aws_rds_mysql.adoc) and [Elasticsearch](https://www.elastic.co/cloud/elasticsearch-service/signup)
- run [this script](https://github.com/confluentinc/learn-kafka-courses/blob/main/data-pipelines/customers.sql) in mysql
- create ratings topic and mysql101.demo.CUSTOMERS entity topic (cleanup compact)
- datagen data in AVRO in that topic
- create mysql cdc source connector:
    - tables included: demo.CUSTOMERS
    - Snapshot mode: when_needed
    - Output message format: AVRO
    - Output messages adter-state only: true
- in Ksqldb create stream rating:
```
CREATE STREAM RATINGS 
  WITH (KAFKA_TOPIC='ratings',VALUE_FORMAT='AVRO');
```
```
SELECT USER_ID, STARS, CHANNEL, MESSAGE 
  FROM RATINGS EMIT CHANGES;
```
- remove test from channel:
```
SELECT USER_ID, STARS, CHANNEL, MESSAGE
  FROM RATINGS
  WHERE LCASE(CHANNEL) NOT LIKE '%test%'
  EMIT CHANGES;
```
```
CREATE STREAM RATINGS_LIVE AS SELECT * FROM RATINGS WHERE LCASE(CHANNEL) NOT LIKE '%test%' EMIT CHANGES;
```
```
SELECT USER_ID, STARS, CHANNEL, MESSAGE
  FROM RATINGS_LIVE
  EMIT CHANGES;
```
- Join in real time (stream) with entity(table):
    - create stream customer:
    ```
    CREATE STREAM CUSTOMERS_S
    WITH (KAFKA_TOPIC ='mysql101.demo.CUSTOMERS',
      KEY_FORMAT  ='JSON',
      VALUE_FORMAT='AVRO');
    ```
    - create table customer from stream:
    ```
    CREATE TABLE CUSTOMERS WITH (FORMAT='AVRO') AS
    SELECT id                            AS customer_id,
           LATEST_BY_OFFSET(first_name)  AS first_name,
           LATEST_BY_OFFSET(last_name)   AS last_name,
           LATEST_BY_OFFSET(email)       AS email,
           LATEST_BY_OFFSET(club_status) AS club_status
    FROM   CUSTOMERS_S
    GROUP BY id;
    ```
    - create stream with join:
    ```
    CREATE STREAM RATINGS_WITH_CUSTOMER_DATA
        WITH (KAFKA_TOPIC='ratings-enriched') AS
    SELECT C.CUSTOMER_ID,
         C.FIRST_NAME + ' ' + C.LAST_NAME AS FULL_NAME,
         C.CLUB_STATUS,
         C.EMAIL,
         R.RATING_ID,
         R.MESSAGE,
         R.STARS,
         R.CHANNEL,
         TIMESTAMPTOSTRING(R.ROWTIME,'yyyy-MM-dd''T''HH:mm:ss.SSSZ') AS RATING_TS
        FROM   RATINGS_LIVE R
        INNER JOIN CUSTOMERS C
          ON R.USER_ID = C.CUSTOMER_ID
        EMIT CHANGES;
    ```
    ```
    SELECT * 
  FROM RATINGS_WITH_CUSTOMER_DATA 
  EMIT CHANGES;
    ```
- if repartition is needed:
```
CREATE STREAM CUSTOMERS_R 
  WITH (PARTITIONS=6) AS 
  select * from  CUSTOMERS_S ;
```
- Make an update in mysql and see live updates
```
UPDATE demo.CUSTOMERS 
   SET CLUB_STATUS='platinum' 
 WHERE ID=1;
```
```
SELECT CUSTOMER_ID, FULL_NAME, CLUB_STATUS, STARS, MESSAGE
  FROM RATINGS_WITH_CUSTOMER_DATA
  WHERE CUSTOMER_ID=1
  EMIT CHANGES;
```
- Lets sink this enriched data (rating-enriched) to elasticsearch with sink connector with AVRO input message format