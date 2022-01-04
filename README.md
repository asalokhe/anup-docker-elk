# ELK + Mule logs Ingestion
Setting up ELK stack inside Docker container and pushing logs from Mule application using log4 Appender "mule-elk-demo" and using to jms and then to Logstash using "mule-mq-demo"

########################################################################################################################################
## prerequisite:
1. Docker
2. docker-compose.
3. Anypoint studio to run the app for data ingestion.
4. Postman or curl.

########################################################################################################################################
## Specifics
| Files | Description |
| ------ | ------ |
| activemq/Dockerfile | To build ActiveMQ server.|
| elasticsearch/config/elasticsearch.yml | Configuration which is part of volume bind path.|
| elasticsearch/Dokerfile | To build Elasticsearch server.|
| kibana/Dockerfile | To build Kibana Server.|
| kibana/config/kibana.yml | Configuration which is part of volume bind path. |
| logstash/Dockerfile | To build logstash server.|
| logstash/pipeline/logstash.conf | Pipeline conf file is used to defining Input-Filter-Output Configuration |
| logstash/config/logstash.yml | Configuration which is part of volume bind path. |

> Note: Configuration files should be mouted and hence needs to be placed in appropriate location as mentioned in docker-compose.yml file.

########################################################################################################################################
## ELK Installation & ActiveMQ installation
    docker-compose up

########################################################################################################################################
## Logstash:

Logstash is configured to listen to two ports, configured in pipeline/logstash.conf file

1. TCP: 5000  
we can configure the TCP connection to inject mule logs in log4j2.xml file as below.


2. JMS: Queue "TestQ" (Can be changed) :
    Logstash will listen to the "TestQ" and will inject all the message from this Queue to Elastic search.
    
########################################################################################################################################
## Mule Application

### mule-elk-demo
Below configuration can be added in the log4j to push the mule application logs; (All the Console logs will be pushed to ELK)

    <!-- sys:elk.host & sys:elk.port can be passed as a system properties "-Delk.host=localhost -Delk.port=5000" to mule application.-->
    <Socket name="socket" host="${sys:elk.host}" port="${sys:elk.port}" protocol="TCP">
        <JsonLayout properties="true" compact="true" eventEol="true"/>
    </Socket>

    <AsyncRoot level="INFO">
        <AppenderRef ref="socket"/>
    </AsyncRoot>

### mule-mq-demo
JMS dependency + push the message in the required queue. 

    <dependency>
        <groupId>org.apache.activemq</groupId>
        <artifactId>activemq-all</artifactId>
        <version>5.16.3</version>
    </dependency>

########################################################################################################################################
## Verification
1. After installation of elk stack and active mq using docker-compose up
2. Run the application "mule-mq-demo" and test; GET: http://localhost:8081/amq to ingest logs via jms.
3. Run the application "mule-jms-demo" and test; GET: http://localhost:8081/elk-demo to ingest console logs using log4j2.xml appender 

########################################################################################################################################
## References:
1. https://www.elastic.co/guide/index.html
2. https://www.elastic.co/guide/en/logstash/current/index.html
3. You can disable CloudHub logs and integrate your CloudHub application with your logging system by using the Log4j configuration. After you configure logs to flow to both your log system and CloudHub, disable the default CloudHub application logs. 
    https://docs.mulesoft.com/runtime-manager/custom-log-appender
4. Logstash beat vs agent: https://www.elastic.co/guide/en/fleet/current/beats-agent-comparison.html

########################################################################################################################################