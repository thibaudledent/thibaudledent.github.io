---
title: "Creating and uploading a Quarkus project to AWS Lambda"
published: true
---

I just wanted to play with Quarkus and AWS Lambda. I did a little proof of concept in one evening. There's a lot of things to improve of course but it shows that Java has evolved a lot and can start really fast. 

This project uses Quarkus, the Supersonic Subatomic Java Framework. If you want to learn more about Quarkus, please visit its website [here](https://quarkus.io/).

# Create the Java project

## Check the last version of the plugin

Check the latest version of the `quarkus-maven-plugin` on [Maven Central - quarkus-maven-plugin](https://search.maven.org/artifact/io.quarkus/quarkus-maven-plugin)

## Create the project using the plugin

```bash
mvn io.quarkus:quarkus-maven-plugin:1.3.0.Final:create \
  -DprojectGroupId=com.github.thibaudledent \
  -DprojectArtifactId=quarkus-poc \
  -DclassName="com.github.thibaudledent.quarkus.poc.HelloResource" \
  -Dpath="/quizz"
```

## Compile in dev mode

```bash
mvn compile quarkus:dev
```

## Try it out with `curl`

```bash
curl -X GET "localhost:8080/quizz"
```

# Run a Containerized Native Image

In order to build a  native image, you don’t need to have GraalVM configured locally, Quarkus can use a dedicated containerized version of GraalVM for that:

## Package the image

```bash
mvn package -Pnative -Dquarkus.native.container-build=true
```

## Build the image

```bash
docker build -f src/main/docker/Dockerfile.native -t thibaudledent/quarkus-poc .
```

A Docker image is created, you can check its size:

```bash
docker images -a | head
```

It outputs:

```bash
REPOSITORY                  TAG     IMAGE ID       CREATED           SIZE
thibaudledent/quarkus-poc   latest  d0d0209d1325   13 seconds ago    155MB
```

## Run the image

```bash
docker run thibaudledent/quarkus-poc
```

Starting time is fast. You can check the startup time with:

```bash
time docker run thibaudledent/quarkus-poc
```

The logs are:

```bash
__  ____  __  _____   ___  __ ____  ______
 --/ __ \/ / / / _ | / _ \/ //_/ / / / __/
 -/ /_/ / /_/ / __ |/ , _/ ,< / /_/ /\ \
--\___\_\____/_/ |_/_/|_/_/|_|\____/___/
2020-03-27 17:23:10,191 INFO  [io.quarkus] (main) quarkus-poc 1.0-SNAPSHOT (powered by Quarkus 1.3.0.Final) started in 0.027s. Listening on: http://0.0.0.0:8080
2020-03-27 17:23:10,191 INFO  [io.quarkus] (main) Profile prod activated.
2020-03-27 17:23:10,191 INFO  [io.quarkus] (main) Installed features: [cdi, resteasy]
^C2020-03-27 17:23:11,081 INFO  [io.quarkus] (main) quarkus-poc stopped in 0.005s
docker run thibaudledent/quarkus-poc  0.03s user 0.02s system 3% cpu 1.669 total
```

## Try it out

```bash
docker exec -it $(docker ps | grep thibaudledent | awk '{print $1}') /bin/bash -c "curl -X GET 'localhost:8080/quizz'"
```

# Running a Standalone Jar

```bash
mvn package
java -jar target/quarkus-poc-1.0-SNAPSHOT-runner.jar
```

# Deploying the Standalone Jar on AWS Lambda

> Note: This step can be improved by using [Quarkus - Amazon Lambda](https://quarkus.io/guides/amazon-lambda) instead.

## Create a function with Java runtime

<img src="https://github.com/thibaudledent/quarkus-poc/raw/master/screenshot_1.png" alt="screenshot_1.png" width="350"/>

([Click to enlarge](https://github.com/thibaudledent/quarkus-poc/raw/master/screenshot_1.png))

## Upload the jar 

Upload the jar from `target/quarkus-poc-1.0-SNAPSHOT-runner.jar` created with `mvn package`:

Use `com.github.thibaudledent.quarkus.poc.HelloResource::hello` as handler

<img src="https://github.com/thibaudledent/quarkus-poc/raw/master/screenshot_2.png" alt="screenshot_2.png" width="350"/>

([Click to enlarge](https://github.com/thibaudledent/quarkus-poc/raw/master/screenshot_2.png))

## Test it

<img src="https://github.com/thibaudledent/quarkus-poc/raw/master/screenshot_3.png" alt="screenshot_3.png" width="350"/>

([Click to enlarge](https://github.com/thibaudledent/quarkus-poc/raw/master/screenshot_3.png))

# References

* The official ["Quarkus"](https://quarkus.io/) website
* ["Quarkus – A New Age of Modern Java Frameworks is Here"](https://4comprehension.com/quarkus-a-new-age-of-modern-java-frameworks-is-here/) (by 4comprehension)
* ["Maven Central - quarkus-maven-plugin"](https://search.maven.org/artifact/io.quarkus/quarkus-maven-plugin) for the Maven plugin
