# anup-docker-elk
Setting up ELK stack inside Docker container. 

Logstash is configured to listen to two ports. 

1. TCP: 5000
we can configure the TCP connection to inject mule logs in log4j2.xml file as below.


    <!-- sys:elk.host & sys:elk.port can be passed as a system properties "-Delk.host=localhost -Delk.port=5000" to mule application.-->
    <Socket name="socket" host="${sys:elk.host}" port="${sys:elk.port}" protocol="TCP">
        <JsonLayout properties="true" compact="true" eventEol="true"/>
    </Socket>

    <AsyncRoot level="INFO">
        <AppenderRef ref="socket"/>
    </AsyncRoot>

2. JMS: Queue "TestQ" (Can be changed)
    Logstash will listen to the "TestQ" and will inject all the message from this Queue to Elastic search.
    