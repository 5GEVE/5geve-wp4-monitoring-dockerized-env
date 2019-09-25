# 5G EVE - WP4 - Monitoring Dockerized environment for testing Monitoring & Data Collection tools

This repository gathers all the tools used for testing the Monitoring & Data Collection features present in the 5G EVE project, within WP4 scope, based on the [Elastic stack](https://www.elastic.co/es/products/). For simulating a complete workflow, it has also been included other tools related to other components from the 5G EVE platform, which are the Data shippers (based on [Filebeat](https://www.elastic.co/es/products/beats/filebeat)), used for collecting the metrics from the monitored components deployed in the site facilities, and the Data Collection Manager (based on [Kafka](https://www.elastic.co/es/products/beats/filebeat)), which gathers the metrics collected by the Data shippers and delivers them to other components interested in using that data, following a publish-subscribe paradigm for that workflow.

The toolchain for the demo is depicted in the following diagram:

![Toolchain for the demo](https://github.com/5GEVE/5geve-wp4-monitoring-dockerized-env/blob/master/resources/images/toolchain.PNG)

And the architecture used for the demo is the following:

![Demo architecture](https://github.com/5GEVE/5geve-wp4-monitoring-dockerized-env/blob/master/resources/images/architecture.PNG)

Note that:

- This demo has been done by using two different VMs: one which contains the [filebeat](https://github.com/5GEVE/5geve-wp4-monitoring-dockerized-env/tree/master/filebeat) repository, and other containing the [kafka-elk](https://github.com/5GEVE/5geve-wp4-monitoring-dockerized-env/tree/master/kafka-elk) repository. However, it can be done in the same server (physical or VM) if needed. In any case, you have to change the <KAFKA_IP> string present in both [filebeat.yml](https://github.com/5GEVE/5geve-wp4-monitoring-dockerized-env/blob/master/filebeat/config/filebeat.yml) and [docker-compose.yml](https://github.com/5GEVE/5geve-wp4-monitoring-dockerized-env/blob/master/kafka-elk/docker-compose.yml) files in order to include the server IP which will hold the [kafka-elk](https://github.com/5GEVE/5geve-wp4-monitoring-dockerized-env/tree/master/kafka-elk) repository.
- The server(s) which will be used for the demo need(s) to have [Docker](https://docs.docker.com/install/linux/docker-ce/ubuntu/) and [Docker Compose](https://docs.docker.com/compose/install/) installed.
- Take care of following the recommendations provided by Elastic regarding the issue of [Running Elasticsearch from the command line in production mode](https://www.elastic.co/guide/en/elasticsearch/reference/master/docker.html#docker-cli-run-prod-mode).
- This demo is a Docker-based adaptation of the following tutorial: [Deploying Kafka with the ELK Stack](https://logz.io/blog/deploying-kafka-with-elk/). Although that tutorial has a slightly different environment, it is a useful reference if you are starting with this set of tools.

## Test stages

Although it is going to be presented the different test stages, with the commands used in each stage, note that in the [/resources/videos folder](https://github.com/5GEVE/5geve-wp4-monitoring-dockerized-env/tree/master/resources/videos) it is available the complete demo recorded in different videos.

### 1. Cleaning up the scenario

```shell
# in the server which will hold the kafka-elk repository
cd kafka-elk
sudo docker-compose down
sudo docker volume prune # if you want to start the demo from zero, deleting all the data saved in Elasticsearch

# in the server which will hold the filebeat repository
cd filebeat
sudo docker container prune
```

### 2. Building the Docker images

```shell
# in the server which will hold the kafka-elk repository
cd kafka-elk
# DO NOT FORGET TO CHANGE <KAFKA_IP> STRING IN docker-compose.yml FILE
sudo docker-compose build

# in the server which will hold the filebeat repository
cd filebeat
# DO NOT FORGET TO CHANGE <KAFKA_IP> STRING IN config/filebeat.yml FILE
sudo docker build -t filebeat .
```

### 3. Deploying the scenario

```shell
# in the server which will hold the kafka-elk repository
cd kafka-elk
sudo docker-compose up
# in a new terminal
sudo docker exec -it kafka /bin/bash
# within the kafka container terminal
	kafka-topics.sh --list --zookeeper 192.168.4.20:2181 # topictest will be printed
	kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic topictest # consume the messages sent to Kafka

# in the server which will hold the filebeat repository
cd filebeat
sudo docker run filebeat

# JSON chains will be sent from filebeat container to kafka container, with the following format:
# {"metrics_series": 1, "resource_uuid": "agv1", "-1": -1, "metrics_data": "-1.15", "metric_name": "deviation", "time": "71.321902"}
```

### 4. Presenting the metrics in Kibana

After logging in Kibana by using a browser (URL: http://<KAFKA_IP>:5601), the first step is to define the Index pattern where the data is being received and saved, which is the Logstash pipeline defined.

![Demo architecture](https://github.com/5GEVE/5geve-wp4-monitoring-dockerized-env/blob/master/resources/images/kibana_1.png)

Then, you can check that data has been received correctly in Elasticsearch.

![Demo architecture](https://github.com/5GEVE/5geve-wp4-monitoring-dockerized-env/blob/master/resources/images/kibana_2.png)

Finally, you can play with the data by generating different graphs. For example, in the following picture, it has been displayed the time-metrics_data graph (average + lower and upper standard deviation).

![Demo architecture](https://github.com/5GEVE/5geve-wp4-monitoring-dockerized-env/blob/master/resources/images/kibana_3.png)
