FROM openjdk:8-jdk-alpine

VOLUME /tmp

ARG JAR_FILE
ARG JAVA_OPTS

ENV JAVA_OPTS=$JAVA_OPTS
ENV PORT=8080

EXPOSE $PORT

COPY ${JAR_FILE} app.jar

# ENTRYPOINT exec java $JAVA_OPTS -jar app-boot.jar
# For Spring-Boot project, use the entrypoint below to reduce Tomcat startup time.
# ENTRYPOINT ["java", "-Dserver.port=${PORT}","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
ENTRYPOINT exec java -Dserver.port=${PORT} $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar app.jar

