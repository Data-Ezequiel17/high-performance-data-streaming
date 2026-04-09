# High performing Kafka and Spark streaming pipeline 
High performing real-time Kafka and Spark streaming pipeline with observability and monitoring tools via prometheus/grafana and elasticsearch/kibana/filebeat

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
