# 100-days-of-Code-with-Apache-Kafka
## Hands on Produce/Consume
- Download and install the latest version in the default directory, ./bin:
    `curl -sL --http1.1 https://cnfl.io/cli | sh -s -- latest`
- Set the PATH environment to include the directory that you downloaded the CLI binaries, ./bin:
    `export PATH=$(pwd)/bin:$PATH`
- Update: `confluent update`
- Login: `confluent login --save`
- list enviroments: `confluent environment list`
- use enviroments: `confluent environment use env-*****`
- list cluster: `confluent kafka cluster list`
- use cluster: `confluent kafka cluster use lkc-*****`
- create api-key: `confluent api-key create --resource lkc-*****`
- use api-ke: `confluent api-key use ********** --resource lkc-*****`
- list topics: `confluent kafka topic list`
- topic from beginning: `confluent kafka topic consume --from-beginning poems`
- produce message: `confluent kafka topic produce poems --parse-key`
```
1:"generations"
2:"weep upon deathstar shadows"
3:"until a ren flies free"
4:"princess Leia"
```

## Hands on Partitioning
- list topics: `confluent kafka topic list`
- describe topic: `confluent kafka topic describe poems`
- create topic 1 partition: `confluent kafka topic create --partitions 1 poems_1`
- describe topic: `confluent kafka topic describe poems_1`
- create topic 4 partitions: `confluent kafka topic create --partitions 4 poems_4`
- describe topic: `confluent kafka topic describe poems_4`
- produce message: `confluent kafka topic produce poems_1 --parse-key`
- produce message: `confluent kafka topic produce poems_4 --parse-key`
```
1:"joins her son in the stars"
2:"final episode"
3:"Han Solo"
4:"sees flashes beyond the moon"
5:"light sabers"
```
## Hands On Consumer
- `sudo apt install python3-pip`
- `sudo apt install python3`
- `pip install confluent-kafka`
- `confluent kafka cluster describe`
- copy host and port from Endpoint
- get your api key and secret put it in config.ini
- `chmod +x consumer.py`
- `./consumer.py config.ini`