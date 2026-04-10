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
 
+ Kibana dashboard to view kafka broker logs
+ Grafana dashboard to view kafka broker health and resource utalization
+ Real-time aggregation and analysis of data
+ Schema registry
  
</details>

<details> 
<summary><strong>Project files description</strong></summary>
 
</details>

## How things work 

Using Docker compose it spins up and connects:
+ Kafka containers comprised of 3 controllers and 3 brokers.
+ Spark contianers comprised of 1 master and 3 workers.
+ Promethius and Grafana containers for kafka broker monotoring(CPU, memory, throughput, ect).
+ Filebeat, ElasticSearch, and Kibana containers for kafka broker logs Observability.
+ Schema registry container to manage and validate data schemas in the pipeline.


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

**Observability** 

Filebeat is a lightweight shipper that monitors specific log files or locations, collects events, and forwards them to a designated output. In this case, ElasticSearch to be indexed. Once these 
logs are indexed in ElasticSearch, Kibana is connected to ElasticSearch to visualize this data to see anything you would like to observe in th elogs(errors, failures, etc).
