# spring-boot

* Run the application in the host OS.
```
gradle build && java -jar ./build/libs/demo-0.0.1-SNAPSHOT.jar
```

* Run the application with the Docker container

Using Docker
```
gradle build docker
docker run -d --name demo1 -p 80:8080 psysanta/demo
```

Using Docker-compose
```
gradle build docker
docker-compose up
```


[REF] https://www.baeldung.com/spring-debugging

# Debugging Spring Applications
## Java Debug Args
First, let’s look at what Java gives us out of the box.

By default, the JVM does not enable debugging. This is because debugging creates additional overhead inside the JVM. It can also be a security concern for applications that are publicly accessible.

Therefore, debugging should only be performed during development and never on production systems.

Before we can attach a debugger, we must first configure the JVM to allow debugging. We do this by setting a command line argument for the JVM:

```
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=8000
```

Let’s break down what each of these values means:

* -agentlib:jdwp

Enable the Java Debug Wire Protocol (JDWP) agent inside the JVM. This is the main command line argument that enables debugging.

* transport=dt_socket

Use a network socket for debug connections. Other options include Unix sockets and shared memory.

* server=y

Listen for incoming debugger connections. When set to n, the process will try to connect to a debugger instead of waiting for incoming connections. Additional arguments are required when this is set to n.

* suspend=n

Do not wait for a debug connection at startup. The application will start and run normally until a debugger is attached. When set to y, the process will not start until a debugger is attached.

* address=8000

The network port that the JVM will listen for debug connections.

The values above are standard and will work for most use cases and operating systems. The JPDA connection guide covers all possible values in more detail.

## Spring Boot Applications
Spring Boot applications can be started several ways. The simplest way is from the command line using the java command with -jar option.

To enable debugging, we would simply add the debug argument using the -D option:

```
java -jar myapp.jar -Dagentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=8000
```

With Maven, we can use the provided run goal to start our application with debugging enabled:

```
mvn spring-boot:run -Dagentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=8000
```

Likewise, with Gradle, we can use the bootRun task. First, we must update the build.gradle file to ensure Gradle passes command line arguments to the JVM:

```
bootRun {
   systemProperties = System.properties
}
```

Now we can execute the bootRun task:

```
gradle bootRun -Dagentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=8000
```

## Application Servers
While Spring Boot has become very popular in recent years, traditional application servers are still quite prevalent in modern software architectures. In this section, we’ll look at how to enable debug for some of the more popular applications servers.

Most application servers provide a script for starting and stopping applications. Enabling debug is typically just a matter of adding additional arguments to this script and/or setting additional environment variables.

### Tomcat
The startup script for Tomcat is named catalina.sh (catalina.bat on Windows). To start a Tomcat server with debug enabled we can prepend jpda to the arguments:

```
catalina.sh jpda start
```

The default debug arguments will use a network socket listening on port 8000 with suspend=n. These can be changed by setting one or more of the following environment variables: JPDA_TRANSPORT, JPDA_ADDRESS, and JPDA_SUSPEND.

We can also get full control of the debug arguments by setting JPDA_OPTS. When this variable is set, it takes precedence over the other JPDA variables. Thus it must be a complete debug argument for the JVM.

### Wildfly
The startup script for Wildfly is stand-alone.sh. To start a Wildfly server with debug enabled we can add –debug.

The default debug mode uses a network listener on port 8787 with suspend=n. We can override the port by specifying it after the –debug argument.

For more control over the debug argument, we can just add the complete debug arguments to the JAVA_OPTS environment variable.

## Debugging From an IDE
https://blog.docker.com/2016/09/java-development-using-docker/

### IntelliJ
...
### Eclipse
...

### VisualStudio Code
### Pre-requisites

* [Docker for OSX, Docker for Windows, or Docker for Linux](https://www.docker.com/products/docker)
* [Visual Studio Code](https://code.visualstudio.com/)
 
launch.json
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "java",
            "name": "Debug (Attach) - Remote",
            "request": "attach",
            "hostName": "localhost",
            "port": "5005",
            "preLaunchTask": "start_docker_compose_debug",
            "postDebugTask": "stop_docker_compose_debug"
        }
    ]
}
```

tasks.json
```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "start_docker_compose_debug",
            "type": "shell",
            "command": "docker-compose",
            "args": [
                "-f",
                "docker/docker-compose.debug.yml",
                "up",
                "-d"
            ]
        },
        {
            "label": "stop_docker_compose_debug",
            "type": "shell",
            "command": "docker-compose",
            "args": [
                "-f",
                "docker/docker-compose.debug.yml",
                "stop"
            ]
        },
        {
            "label": "docker_build",
            "type": "shell",
            "command": "gradle",
            "args": [
                "build",
                "docker"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}
```

## Debugging with Docker
Debugging a Spring application inside a Docker container may require additional configuration. If the container is running locally and is not using host network mode, then the debug port will not be accessible outside the container.

There are several ways to expose the debug port in Docker.

We can use –expose with the docker run command:
```
docker run --expose 8080 psysanta/demo
```

We can also add the EXPOSE directive to the Dockerfile:

```
EXPOSE 8000
```

Or if we’re using Docker Compose, we can add it into the YAML:

```
expose:
 - "8000"
```

