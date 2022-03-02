# confluentinc-arm64
Custom build for arm CPU architecture like Apple Silicon of confluentinc Docker images.

# Setup and prerequisites
1. `brew install socat`
2. `brew install openjdk` and set JAVA_HOME and add JAVA_HOME/bin to PATH
3. `brew install maven`
4. Docker Desktop for M1: https://docs.docker.com/desktop/mac/apple-silicon/


# Expose the Docker control socket for Maven
```shell

# Expose the Docker control socket on 2375 for Maven
socat TCP-LISTEN:2375,reuseaddr,fork UNIX-CONNECT:/var/run/docker.sock
export DOCKER_HOST=tcp://127.0.0.1:2375
```

# Build the base image

1. Get the base image (only the first time)
```shell

## Set up the subtree initially.
git subtree add --prefix docker-common https://github.com/confluentinc/common-docker.git master --squash

## Pull newer versions
git subtree pull --prefix docker-common https://github.com/confluentinc/common-docker.git master --squash
  
```

2. Add the confluent repository to the `pom.xml` file
```xml
<repositories>
     <repository>
         <id>confluent</id>
         <url>https://packages.confluent.io/maven/</url>
     </repository>
 </repositories>
```

4. Change the property ubi.openssl.version in the pom.xml to 1.1.1k (check the website for the latest version)
5. Change the property ubi.zulu.openjdk.version in the pom.xml to 11.0.14 (check the website for the latest version)
6. Build the image
```shell
mvn clean package \
  -DskipTests -Pdocker \
  -Ddocker.registry=nxt/
```


3. Build Kafka and Zookeeper images
```shell

```

# Reference
https://nxt.engineering/blog/kafka-docker-image/

