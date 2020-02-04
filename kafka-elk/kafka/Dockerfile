FROM wurstmeister/kafka

RUN apk add --no-cache nano htop

CMD["kafka-topics.sh --create --zookeeper 10.9.8.203:2181 --replication-factor 1 --partitions 1 --topic topic_1"]
CMD["kafka-topics.sh --create --zookeeper 10.9.8.203:2181 --replication-factor 1 --partitions 1 --topic topic_2"]
CMD["kafka-topics.sh --create --zookeeper 10.9.8.203:2181 --replication-factor 1 --partitions 1 --topic topic_3"]
