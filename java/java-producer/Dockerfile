ARG BUILD_HOME=/app

# Builder ############################################
FROM maven:3.6.0-jdk-11-slim AS builder

RUN  apt-get update \
  && apt-get install -y wget \
  && rm -rf /var/lib/apt/lists/*

ARG OTEL_VERSION=1.15.0 
 

ENV APP_HOME=$BUILD_HOME
WORKDIR $APP_HOME  

RUN wget https://github.com/open-telemetry/opentelemetry-java-instrumentation/releases/download/v${OTEL_VERSION}/opentelemetry-javaagent.jar 

COPY metrictest.groovy $APP_HOME/metrictest.groovy

COPY src $APP_HOME/src
COPY config $APP_HOME/config
COPY pom.xml $APP_HOME
RUN mvn -f $APP_HOME/pom.xml clean package  

# App ############################################
FROM adoptopenjdk/openjdk11:ubi  

COPY --from=builder $APP_HOME/target/java-client-jar-with-dependencies.jar app.jar
COPY --from=builder $APP_HOME/config/application-docker.yml application-docker.yml 
 
COPY --from=builder $APP_HOME/opentelemetry-javaagent.jar opentelemetry-javaagent.jar

EXPOSE 38080
  
EXPOSE 5000 

ENV COLLECTOR_ENDPOINT="http://collector:4317"
ENV APP_NAME="java-producer"  

ENV JAVA_TOOL_OPTIONS "-Dcom.sun.management.jmxremote \
   -Dcom.sun.management.jmxremote.ssl=false \
  -Dcom.sun.management.jmxremote.authenticate=false \
  -Dcom.sun.management.jmxremote.port=5000 \
  -Dcom.sun.management.jmxremote.rmi.port=5000 \
  -Dcom.sun.management.jmxremote.host=0.0.0.0 \
  -Djava.rmi.server.hostname=java-producer \
  -Dspring.profiles.active=docker \ 
  -javaagent:opentelemetry-javaagent.jar \
  -Dotel.traces.exporter=otlp \
  -Dotel.metrics.exporter=otlp \ 
  -Dotel.service.name=${APP_NAME} \
  -Dotel.exporter.otlp.endpoint=${COLLECTOR_ENDPOINT}"

CMD ["java", "-jar", "app.jar","application-docker.yml"]

