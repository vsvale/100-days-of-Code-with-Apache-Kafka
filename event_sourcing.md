- create a ksqldb cluster: `shopping_cart_database`
- offset: `earliest`
- add another field: `ksql.streams.commit.interval.ms = 3000`
- create shopping_cart_events:
```
CREATE STREAM shopping_cart_events (customer VARCHAR, item VARCHAR, qty INT)
WITH (kafka_topic='shopping_cart_events', value_format='json', partitions=1);
```
```
--add two pairs of pants
INSERT INTO shopping_cart_events (customer, item, qty)
VALUES ('bob', 'pants', 2);
--add a t-shirt
INSERT INTO shopping_cart_events (customer, item, qty)
VALUES ('bob', 't-shirts', 1);
--remove one pair of pants
INSERT INTO shopping_cart_events (customer, item, qty)
VALUES ('bob', 'pants', -1);
--add a hat
INSERT INTO shopping_cart_events (customer, item, qty)
VALUES ('bob', 'hats', 1);	
```
```
SELECT * FROM shopping_cart_events EMIT CHANGES;
```
- create a summary table:
```
CREATE TABLE current_shopping_cart WITH (KEY_FORMAT='JSON') AS
  SELECT customer, item, SUM(qty) as total_qty 
  FROM   shopping_cart_events 
  GROUP BY customer, item 
  EMIT CHANGES;
```
```
SELECT * FROM current_shopping_cart EMIT CHANGES;
```