## Hands On: Getting Started with Kafka Connect
- create cluester in confluent
- Data integration-> connnector -> Datagen
- create orders topic and launch connector

## Hands On: Use SMTs with a Managed Connector
- On Datagen go to advanced option and transforms
- Add an SMT:
    - Transform name: castValues
    - transform type: org.apache.kafka.connect.transforms.Cast$Value
    - spec: orderid:string, orderunits:int32
- add another SMT:
    - Transform name: tsConverter
    - transform type: org.apache.kafka.connect.transforms.TimestampConverter$Value
    - target.type: string
    - field: odertime
    - format: yyyy-MM-dd

## Hands On: Confluent Cloud Managed Connector API
- connect to confluent CLI
- `confluent schema-registry cluster enable --cloud gcp --geo us`
- `confluent api-key create  --resource <SR cluster ID>`
- `confluent kafka client-config create java --sr-apikey <sr API key> --sr-apisecret <sr API secret> | tee $HOME/.confluent/java.config`
```
(cd ~ && \
rm -rf delta_configs && \
source ~/ccloud_library.sh && \
ccloud::generate_configs $HOME/.confluent/java.config)
```




- `CLOUD_AUTH64=$(echo -n <API KEY>:<API SECRET> | base64 -w0)`
- `CLUSTER_AUTH64=$(echo -n $CLOUD_KEY:$CLOUD_SECRET | base64 -w0)`
 ```
curl --request GET \
--url 'https://api.confluent.cloud/org/v2/environments' \
--header 'Authorization: Basic '$CLOUD_AUTH64''
```