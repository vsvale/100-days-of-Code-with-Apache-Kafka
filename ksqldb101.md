### create stream
`CREATE STREAM MOVEMENTS (PERSON VARCHAR KEY, LOCATION VARCHAR)
  WITH (VALUE_FORMAT='JSON', PARTITIONS=1, KAFKA_TOPIC='movements');`

#### stream vs table
Steam: append all events, unbound stream of events
Table: current state for given key

### insert into stream
```
INSERT INTO MOVEMENTS VALUES ('Allison', 'Denver');
INSERT INTO MOVEMENTS VALUES ('Robin', 'Leeds');
INSERT INTO MOVEMENTS VALUES ('Robin', 'Ilkley');
INSERT INTO MOVEMENTS VALUES ('Allison', 'Boulder');
```

#### See all data
`SET 'auto.offset.reset' = 'earliest';`

#### Select data from movements stream
`SELECT * FROM MOVEMENTS EMIT CHANGES;`

#### create a table
```
CREATE TABLE PERSON_STATS WITH (VALUE_FORMAT='AVRO') AS
  SELECT PERSON,
    LATEST_BY_OFFSET(LOCATION) AS LATEST_LOCATION,
    COUNT(*) AS LOCATION_CHANGES,
    COUNT_DISTINCT(LOCATION) AS UNIQUE_LOCATIONS
  FROM MOVEMENTS
GROUP BY PERSON
EMIT CHANGES;
```

#### create a source connector
```
CREATE SOURCE CONNECTOR SOURCE_MYSQL_01
WITH (
  'connector.class'= 'MySqlConnector',
  'database.hostname' = 'mysql'
  'table.whitelist' = 'demo.customers'
);
```

### create a datagen source connector
```
CREATE SOURCE CONNECTOR datagen_clickstream_users WITH (
  'name'                     = 'DATAGEN_USERS',
  'connector.class'          = 'DatagenSource',
  'kafka.api.key'            = '<my-kafka-api-key>',
  'kafka.api.secret'         = '<my-kafka-api-secret>',
  'kafka.topic'              = 'clickstream_users',
  'quickstart'               = 'CLICKSTREAM_USERS',
  'maxInterval'              = '10',
  'tasks.max'                = '1',
  'output.data.format'       = 'JSON'
);
```

### create sink connector
```
 CREATE SINK CONNECTOR analyzed_clickstream WITH (
   'connector.class'          = 'ElasticsearchSink',
   'name'                     = 'recipe-elasticsearch-analyzed_clickstream',
   'input.data.format'        = 'JSON',
   'kafka.api.key'            = '<my-kafka-api-key>',
   'kafka.api.secret'         = '<my-kafka-api-secret>',
   'topics'                   = 'USER_IP_ACTIVITY, ERRORS_PER_MIN_ALERT',
   'connection.url'           = '<elasticsearch-URI>',
   'connection.username'      = '<elasticsearch-username>',
   'connection.password'      = '<elasticsearch-password>',
   'type.name'                = 'type.name=kafkaconnect',
   'tasks.max'                = '1',
   'key.ignore'               = 'true',
   'schema.ignore'            = 'true'
 );
 ```

 ### filter
 ```
 CREATE STREAM ORDERS_NY AS
  SELECT *
    FROM ORDERS
   WHERE ADDRESS->STATE='New York';
 ```

 ### join
 ```
 CREATE STREAM user_clickstream AS
  SELECT
    u.user_id,
    u.username,
    ip,
    u.city,
    request,
    status,
    bytes
  FROM clickstream c
  LEFT JOIN web_users u ON cast(c.userid AS VARCHAR) = u.user_id;
 ```

 ### transform
 ```
  CREATE STREAM ORDERS_NO_ADRESS_DATA AS
  SELECT TIMESTAMPTOSTRING(ROWTIME,'yyyy-MM-dd HH:mm:ss') AS ORDER_TIMESTAMP,
  ORDERID, ITEMID, ORDERUNITS
  FROM ORDERS;
 ```

 ### nested fields
 ```
 CREATE STREAM ORDERS_FLAT AS
  SELECT TIMESTAMPTOSTRING(ORDERTIME, 'yyyy-MM-dd HH:mm:ss') AS ORDER_TIMESTAMP,
         ORDERID,
         ITEMID,
         ORDERUNITS,
         ADDRESS->STREET AS ADDRESS_STREET,
         ADDRESS->CITY   AS ADDRESS_CITY,
         ADDRESS->STATE  AS ADDRESS_STATE
    FROM ORDERS;
 ```

 ### mirror to CSV stream
 ```
 CREATE STREAM source_csv_stream (ITEM_ID INT, 
                                 DESCRIPTION VARCHAR, 
                                 UNIT_COST DOUBLE, 
                                 COLOUR VARCHAR, 
                                 HEIGHT_CM INT, 
                                 WIDTH_CM INT, 
                                 DEPTH_CM INT) 
                          WITH (KAFKA_TOPIC ='source_topic', 
                                VALUE_FORMAT='DELIMITED');
 ```
 ```
CREATE STREAM orders_csv
WITH(VALUE_FORMAT='delimited', KAFKA_TOPIC='orders_csv') AS
SELECT * FROM orders_flat EMIT CHANGES;
 ```

 ### Merge streams
 ```
 INSERT INTO ORDERS_COMBINED
  SELECT 'UK' AS SOURCE,
         CONCAT_WS('-','UK',CAST(ORDERID AS VARCHAR)) AS ORDERID,
         ORDERTIME,
         ITEMID,
         ORDERUNITS,
         ADDRESS
    FROM ORDERS_UK
    PARTITION BY CONCAT_WS('-','UK',CAST(ORDERID AS VARCHAR));
 ```
 ```
 CREATE STREAM orders_combined AS
SELECT 'US' AS source, ordertime, orderid, itemid, orderunits, address
FROM orders_us;
INSERT INTO orders_combined
SELECT 'UK' AS source, ordertime, orderid, itemid, orderunits, address
FROM orders_uk;
 ```

 ### splitting streams
 ```
 CREATE STREAM ORDERS_US AS
   SELECT * FROM ORDERS_COMBINED
   WHERE SOURCE = 'US';

CREATE STREAM ORDERS_UK AS
   SELECT * FROM ORDERS_COMBINED
   WHERE SOURCE = 'UK';

CREATE STREAM ORDERS_OTHER AS
   SELECT * FROM ORDERS_COMBINED
   WHERE SOURCE != 'US'
   AND SOURCE != 'UK';
 ```

### Aggregations
- count, cont distinct, sum, average, min, max
- always return table, with the key being filds in group by clause
```
SELECT PERSON, COUNT (*)
FROM MOVEMENTS GROUP BY PERSON EMIT CHANGES;
SELECT PERSON, COUNT_DISTINCT(LOCATION)
FROM MOVEMENTS GROUP BY PERSON EMIT CHANGES;
CREATE TABLE PERSON_STATS AS
SELECT PERSON,
		LATEST_BY_OFFSET(LOCATION) AS LATEST_LOCATION,
		COUNT(*) AS LOCATION_CHANGES,
		COUNT_DISTINCT(LOCATION) AS UNIQUE_LOCATIONS
	FROM MOVEMENTS
GROUP BY PERSON
EMIT CHANGES;
```

### Push and pull queries
pull query to get current state of the data for a given key. The result is returned and the query terminates. Only avaiable in tables.
Push query "emit changes", this query will continue running and emit the results of the query for any changes thar occur in the underlying stream

#### Map, reduce & filter
```
CREATE STREAM transformed 
    AS SELECT id, name,
    TRANSFORM(exam_scores,(k, v) => UCASE(k), (k, v) => (ROUND(v))) AS rounded_scores
FROM stream1 EMIT CHANGES;
```
```
CREATE STREAM reduced 
    AS SELECT name,
    REDUCE(points,0,(s,x)=> (s+x)) AS total
FROM stream2 EMIT CHANGES;
```
```
CREATE STREAM filtered 
    AS SELECT id,
    FILTER(numbers,x => (x%2 = 0)) AS even_numbers
FROM stream3 EMIT CHANGES;
```