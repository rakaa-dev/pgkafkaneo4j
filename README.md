# pgkafkaneo4j

PREPARATION Kafka (Confluent)

Download confluent.7.0.1.tar.gz
$ wget https://packages.confluent.io/archive/7.0/confluent-7.0.1.tar.gz

Extract the confluent tar.gz file that has been downloaded
$ tar -xvf confluent-7.0.1.tar.gz confluent-7.0.1/

Export path confluent
$ export CONFLUENT_HOME=/home/ifgpoc1/confluent-7.0.1
$ export PATH=$PATH:$CONFLUENT_HOME/bin

Install the datagen plugin for kafka connect
$ confluent-hub install --no-prompt confluentinc/kafka-connect-datagen:latest

Install plugin Kafka Connect Neo4j
$ confluent-hub install neo4j/kafka-connect-neo4j:latest

Install Debezium Connector Postgresql
$ confluent-hub install debezium/debezium-connector-postgresql:latest

Start all services confluent
$ confluent local services start

***PostgreSQL Database***

To setup Postgresql we need the Wal2json plugin for the Debezium connector to be able to convert Postgres output into JSON published to Kafka topic..
$ sudo apt-get install postgresql-12-wal2json

Config postgresql
$ su - postgres
$ nano /etc/postgresql/12/main/postgresql.conf
Uncomment listen_addresses = '*'

$ nano /etc/postgresql/12/main/pg_hba.conf

Create table
create table PESERTA(NOPESERTA text,NAMAPESERTA text,NOPEG text,TGLLAHIR text,ALAMAT1 text,ALAMAT2 text,KOTA text,KDPROPINSI text,KODEPOS text,TELP text ,FAX text,EMAIL text ,MARITALSTATUS text,STATUS text,TGLSTATUS text,KDJABATAN text,KDDIVISI text,TEMPATLAHIR text,KDPEKERJAAN text,KDGOLONGAN text,NOCUSTOMER text,JK text,KDSTATUSMEDICAL text,TGLREKAM text,USERREKAM text,TGLUPtext text,USERUPtext text,NOSERTIFIKAT text,ESCOBAR text,NOHP text,NOID text,JNSID text,NEWESCOBAR text,KDUNIT text,STAKOREKSI text,AUTODEBET text,NOPENGAJUAN text);

Set full identity replication on the newly created table.
ALTER TABLE public.peserta REPLICA IDENTITY FULL

Convert file format to utf8 and insert data to table 
iconv -c -t utf8 PESERTA.txt > PESERTA1.txt
copy peserta from '/home/ifgpoc1/NON_EBP/PESERTA1.txt' with delimiter '|';


**Kafka Debezium Connector

Back to ifgpoc user  and create file debezium-connector.json

 $ nano debezium-connector.json
Create config:

 {
    "name": "ifgtesting1-connector",
    "config": {
        "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
        "tasks.max": "1",
        "plugin.name": "wal2json",
        "database.hostname": "id-ifg-poc-kafka4",
        "database.port": "5432",
        "database.user": "postgres",
        "database.password": "postgres",
        "database.dbname": "postgres",
        "database.server.name": "id-ifg-poc-kafka4",
        "key.converter": "org.apache.kafka.connect.json.JsonConverter",
        "value.converter": "org.apache.kafka.connect.json.JsonConverter",
        "key.converter.schemas.enable": "false",
        "value.converter.schemas.enable": "false",
        "snapshot.mode": "always"
    }
}

$ curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ --data @debezium-connector.json 
Then the output of the post is auto create topic and auto scan all tables in postgres.

$ curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ --data @pgkafkaneo4j.json
















   

