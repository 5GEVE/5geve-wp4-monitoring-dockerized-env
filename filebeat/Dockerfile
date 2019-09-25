FROM docker.elastic.co/beats/filebeat:7.3.2
COPY config/filebeat.yml /usr/share/filebeat/filebeat.yml
COPY files/asti-metrics-json-chains.txt /tmp/asti-metrics-json-chains.txt
USER root
RUN chown root:filebeat /usr/share/filebeat/filebeat.yml
RUN chmod go-w /usr/share/filebeat/filebeat.yml
USER filebeat
