# High performing Kafka and Spark streaming pipeline 
High performing real-time Kafka and Spark streaming pipeline with observability and monitoring tools via prometheus/grafana and elasticsearch/kibana/filebeat

<img width="1595" height="1195" alt="image" src="https://github.com/user-attachments/assets/c3d88287-a1d0-4eb2-b2f6-121514326ab9" />

<details> 
<summary><strong>Technologies</strong></summary>
 
+ Python
+ Kafka
+ Docker
+ Spark streaming
+ Promethius
+ Grafana
+ ElasticSearch
+ Filebeat
+ Kibana
+ Confluent schema registry
 
</details>

<details> 
<summary><strong>About the project</strong></summary>
 
All within Docker this project mimicks a production level real-time pipeline. Fake bank transaction data is generated and fed into a kafka topic where it is then sent to Spark via 
spark streaming to aggregate and analyze the data in real-time. 

A kafka KRaft cluster made up of 3 contollers and 3 brokers was used instead of one kafka controller/broker to mimick 
the real world scenario of distributed systems. I wanted to test the fault tolorence of kafka when one broker or controller fails, where if the leader fails then a new one is 
voted in by the followers via controller election. I also used 3 spark workers and a master for the spark streaming cluster for similar reasons. 

I threw in Promethius and Grafana containers for monotoring the health and resources of the kafka brokers as you would a production pipeline.

I also used Filebeat, ElasticSearch and Kibana for observability and visualization of the Kafka broker logs as you would in a production pipeline. 
 
</details>

<details> 
<summary><strong>Features</strong></summary>
 
+ Kibana dashboard to view kafka controller and broker logs for observability
+ Grafana dashboard to monitor kafka broker health and resource utalization
+ Real-time aggregation and analysis of data with spark streaming
+ Schema registry
  
</details>

<details> 
<summary><strong>Project files description</strong></summary> 


+ **docker-compose.yml** -YAML configuration file used by Docker Compose to define and manage multi-container Docker applications.
+ **main.py** -python script that creates fake transaction data and send it to kafka.
  
jobs folder
+ **spark_processor.py** -python file that is ran using spark submit to connect/ingest/process data from kafka into spark streaming.

monitoring folder
+ **alert_rules.yml** -define Prometheus alerting rules to trigger notifications when specific conditions are met.
+ **prometheus.yml** -main prometheus config file to format to define how the server scrapes metrics, processes rules, and sends alerts.
+ **filebeat.yml** -main config file defining where to find logs (inputs), how to process/parse them (processors), and where to send them, such as Elasticsearch.
+ **prometheus_ds.yml** -config file to connect prometheus to grafana.
+ **logstash.conf** -used by Logstash to define how data should be ingested, transformed, and sent to its final destination.
 
volumes folder
+ **jmx_prometheus_javaagent-1.0.1.jar** -a Java agent from the Prometheus JMX Exporter project. It attaches to Java Virtual Machines (JVM) 
to collect JMX MBean metrics and expose them in a Prometheus-compatible format. You can get at https://github.com/prometheus/jmx_exporter/releases
+ **kafka-broker.yml** -config file that tells jmx_exporter what/how you want to be extracted from the kafka brokers and controllers.

</details>

## How things work 

Using Docker compose it spins up and connects:
+ Kafka containers comprised of 3 controllers and 3 brokers.
+ Spark contianers comprised of 1 master and 3 workers.
+ Promethius and Grafana containers for kafka broker monotoring(CPU, memory, throughput, ect).
+ Filebeat, ElasticSearch, and Kibana containers for kafka broker logs Observability.
+ Schema registry container to manage and validate data schemas in the pipeline.
+ Alert manager container to handle alerts sent by client applications such as the Prometheus server in this case.

**Core logic** 

Inside a python virtual environment a python script generates fake bank transaction data and acts as a kafka producer via confluent_kafka library. 
Once this data is in a Kafka topic it is streamed to spark for aggregations via readStream API where it is deserialized and converted into a dataframe to be processed/aggregated using pyspark or sparksql.
Once the data is processed/aggregated it can be sent out to another place like S3 for storage, tableau/powerbi for data visualization, or even elastic search for indexing. However, i decided 
to send it back to kafka inside an aggregated topic. This is common practice to lessen the load on the spark cluster and serves better to send the aggregated data to kafka because kafka can
distribute this data to various places better than spark can.   


**Monitoring** 

I configured log4j via commands in docker compose file to format the logs from the kafka brokers and controllers. JMX_Exporter is used and works with Log4j by scraping the internal management beans (MBeans) 
that Log4j naturally exposes to the Java Virtual Machine (JVM). This allows you to convert Log4j's internal state—such as log levels, configuration details, and appender performance—into a format that monitoring tools like Prometheus can understand.
once the data(logs) is in Promethius then Grafana is used to connect to promethius and visualize the data with dashboards.

In Grafana every dashboard is a single JSON document that contains all the settings, panel configurations, and data queries to create that dashboard.
You can customize the JSON to modify and create any dashboard you want to see any metric you want for the system you are monitoring. Kafka controllers and brokers in our case.
You can create a JSON document from scratch or get one from grafana.com

**Observability** 

Logstash is a server-side data processing tool that ingests data from multiple sources simultaneously, transforms it, and sends it to a "stash" like Elasticsearch to be indexed. Once these logs are indexed in ElasticSearch, Kibana is connected to ElasticSearch to visualize this data to see anything you would like to observe in the logs(errors, failures, etc).


## How to run
**prequalifications:**
+ have docker desktop installed and running
+ have a powerful setup(32GB ram+/ 9i). If this doesnt work on your personal then use a powerful ec2 instance

**STEP 1** - create a python virtual environment in your IDE by running these two commands in terminal: 
```bash 
python -m venv venv
```
```bash 
venv\Scripts\activate
```

**STEP 2** - once virtual environment is running, run pip install command to install confluent_kafka and pyspark:
```bash
 pip install confluent_kafka 
```
```bash
 pip install pyspark
```

**STEP 3** - once that is done, run docker compose up:
```bash 
docker compose up -d
```

**STEP 4** - open two terminals, possibly 3 to run seperate commands. In one terminal run the main.py python script to generate fake transaction data into kafka. In the other terminal run the 'docker exec' command to run a spark submit in the master spark container to run the spark_processor.py python script to connect/ingest/process data from kafka into spark streaming. 

terminal 1 run: 
```bash 
python main.py
```

terminal 2 run:
```bash 
docker exec -it kafkaspark-spark-master-1 spark-submit --master spark://spark-master:7077 --packages org.apache.spark:spark-sql-kafka-0-10_2.12:3.5.0 jobs/spark_processor.py
```

**STEP 5** - you can go to kafka ui to check kafka topics and other info. You can go to master spark ui to view the spark worker health/metrics and spark job currently running.

kafka ui - http://localhost:8080/

spark ui - http://localhost:4040/ 

----------------------------------------------------------------------------------------------------------------------------------------------
\
**How to view monitoring w/ Promethius and Grafana**

**STEP 1** - go to grafana dashboard on http://localhost:3000/ and log in username: admin password: admin. It will ask you to change password. Chnage it to whatever you want.

**STEP 2** - In grafana go to dashboards and click 'NEW' then 'IMPORT'. Enter the dashboard ID '24626' in the second to last box and click 'LOAD'. Then at bottom select 'DS_PROMETHEUS' and then 'IMPORT'. Go to dashboard tab and you should see all the metrics available.

----------------------------------------------------------------------------------------------------------------------------------------------
\
**How to view observability w/ ElasticSearch and Kibana**

**STEP 1** - go to Kibana dashboard on http://localhost:5601/ and click 'Add integrations' and type 'logstash' in search bar. Click 'logstash logs' and at the bottom click 'logstash logs'. Then 'create data view'.

**STEP 2** - under name type whatever name you want. under index pattern type 'kafka-logs-logstash-'. under timestamp field pick @timestamp.

**STEP 3** - click 'view available dashboards' and then 'create a dashboard' and then 'create visualization'. You can now create your own custom observability dashboard by drag and drop fiels on the left hand side.

ElasticSearch - http://localhost:9200/


<details> 
<summary><strong>picture</strong></summary>
 
<img width="2780" height="1417" alt="image" src="https://github.com/user-attachments/assets/2caccc62-fee7-40a9-8b33-6840df034fdc" />

</details>

----------------------------------------------------------------------------------------------------------------------------------------------
\
**TO TERMINATE PROGRAMS** 

+ to end the generate_data.py script: \
```Ctrl+C``` or end terminal

+ to shut down docker containers:
```bash
docker compose down -v
```
+ to end python virtual environment:
```bash
deactivate
```
