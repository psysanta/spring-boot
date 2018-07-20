# spring-boot

* Rrun the application in the host OS.
```
gradle build && java -jar ./build/libs/demo-0.0.1-SNAPSHOT.jar
```

* Rrun the application with the Docker container
```
gradle build docker
docker run -d --name demo1 -p 80:8080 psysanta/demo
```

# Debugging the application in a Docker container