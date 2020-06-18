# Debezium on Openshift 4 with Strimzi

## Preparation: Install mysql from debezium example
Login to Openshift CLI and create new project
```bash
$ oc login
$ oc new-project erfin-debezium
```
Deploy mysql database from debezium's example
```bash
$ oc new-app docker.io/debezium/example-mysql:0.5 \
-e MYSQL_ROOT_PASSWORD=debezium \
-e MYSQL_USER=mysqluser \
-e MYSQL_PASSWORD=mysqlpw
```

## Step 1: Install Strimzi Kafka Cluster using Openshift web console
This steps need **cluster-admin** role to install Strimzi Operator to your namespace (erfin-debezium)

## Step 2: Install Strimzi Kafka Connect Source to Image (s2i) using Openshift web console 
Install strimzi Connect s2i

Expose route Kafka connect api to register database's cdc listener later
```bash
$ oc expose svc/my-connect-cluster-connect-api
$ oc get route
```
result
```bash
NAME                             HOST/PORT                                                                                  
my-connect-cluster-connect-api   my-connect-cluster-connect-api-erfin-debezium.apps.erfin-cluster.sandbox1459.opentlc.com 
```

## Step 3: Create folder for plugins Download Debezium MySql Connector on your Local computer
```bash
$ mkdir plugins
$ cd plugins
$ wget https://repo1.maven.org/maven2/io/debezium/debezium-connector-mysql/1.1.2.Final/debezium-connector-mysql-1.1.2.Final-plugin.tar.gz
$ tar -xzf debezium-connector-mysql-1.1.2.Final-plugin.tar.gz
```
Structure will be like this
```bash
$HOME/plugins
-|
 |-debezium-connector-mysql
   |- ...
   |- debezium-connector-mysql-1.1.2.Final.jar
   |- ...
```
## Step 4: Upload debezium plugin to Kafka Connect
```bash
$ oc start-build my-connect-cluster-connect --from-dir $HOME/plugins/
```
## Step 5: Register debezium connector to target DB (example-mysql) using REST API 
```bash
$ oc get route
$ curl -i -X POST \
-H "Accept:application/json" \
-H "Content-Type:application/json" \
http://my-connect-cluster-connect-api-erfin-debezium.apps.erfin-cluster.sandbox1459.opentlc.com/connectors \
-d \
'{ "name": "inventory-connector", \
"config": { \
"connector.class": "io.debezium.connector.mysql.MySqlConnector", \
"tasks.max": "1", \
"database.hostname": "example-mysql", \
"database.port": "3306", \
"database.user": "debezium", \
"database.password": "dbz", \
"database.server.id": "184054", \
"database.server.name": "dbserver1", \
"database.whitelist": "inventory", \
"database.history.kafka.bootstrap.servers": "my-cluster-kafka-brokers:9092", \
"database.history.kafka.topic": "dbhistory.inventory" } }'
```
Result if success
```
201Created
```
Or use Postman for simpler use :)

## Try your application!
Deploy kafka client apps using Knative. Note: this application use quarkus with smallrye kafka client on my Quay.io registry :)
``` bash
$ oc new-app quay.io/efeluzy/quarkus-kafka-consumer:latest \
-e mp.messaging.incoming.mytopic-subscriber.topic=dbserver1.inventory.customers \
$ curl http://quarkus-kafka-consumer-erfin-debezium.apps.erfin-cluster.sandbox1459.opentlc.com/stream
```

### Bonus: deploy your apps as knative serverless application.
```bash
$ kn service update quarkus-kafka-consumer \
--image quay.io/efeluzy/quarkus-kafka-consumer:latest \
--revision-name quarkus-kafka-consumer-v4 \
--env mp.messaging.incoming.mytopic-subscriber.topic=dbserver1.inventory.customers
$ kn route list
$ curl http://quarkus-kafka-consumer-erfin-debezium.apps.erfin-cluster.sandbox1459.opentlc.com/stream
```


