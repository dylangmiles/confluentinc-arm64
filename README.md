# confluentinc-arm64
Custom build for arm CPU architecture like Apple Silicon of confluentinc Docker images.

## Reference
https://nxt.engineering/blog/kafka-docker-image/

## Setup and prerequisites
1. `brew install socat`
2. `brew install openjdk` and set JAVA_HOME and add JAVA_HOME/bin to PATH
3. `brew install maven`
4. Docker Desktop for M1: https://docs.docker.com/desktop/mac/apple-silicon/


## Expose the Docker control socket for Maven
```shell

# Expose the Docker control socket on 2375 for Maven
socat TCP-LISTEN:2375,reuseaddr,fork UNIX-CONNECT:/var/run/docker.sock
export DOCKER_HOST=tcp://127.0.0.1:2375
```

## Build the base image

1. Get the base image (only the first time)
```shell

## Set up the subtree initially.
git subtree add --prefix docker-common https://github.com/confluentinc/common-docker.git tags/v7.0.2 --squash

## Pull newer versions
git subtree pull --prefix docker-common https://github.com/confluentinc/common-docker.git tags/v7.0.2 --squash
  
```

2. Add the confluent repository to the `docker-common/pom.xml` file

```xml
<repositories>
     <repository>
         <id>confluent</id>
         <url>https://packages.confluent.io/maven/</url>
     </repository>
 </repositories>
```

4. Change the property ubi.openssl.version in the pom.xml to 1.1.1k (check the website for the latest version: https://rpmfind.net/linux/rpm2html/search.php?query=openssl&submit=Search+...&system=centos&arch=aarch64) 
5. Change the property ubi.zulu.openjdk.version in the pom.xml to 11.0.14 (check the website for the latest version: https://www.azul.com/downloads/?package=jdk#download-openjdk)
6. Build the base image
```shell
cd docker-common
mvn clean package \
  -DskipTests -Pdocker \
  -Ddocker.registry=nxt/
```


## Build Kafka and Zookeeper images
1. Get the Kafka images repositiory
```shell

## Set up the subtree initially.
git subtree add --prefix kafka-images https://github.com/confluentinc/kafka-images tags/v7.0.2 --squash

## Pull newer versions
git subtree pull --prefix kafka-images https://github.com/confluentinc/kafka-images tags/v7.0.2 --squash
  
```

2. Add the confluent repository to the `kafka-images/pom.xml` file

```xml
<repositories>
     <repository>
         <id>confluent</id>
         <url>https://packages.confluent.io/maven/</url>
     </repository>
 </repositories>
```

3. Build the kafka-images 
```shell
cd kafka-images
mvn clean package \
  -DskipTests -Pdocker \
  -DCONFLUENT_PACKAGES_REPO='https://packages.confluent.io/rpm/7.0' \
  -DCONFLUENT_VERSION=7.0.2 \
  -Ddocker.registry=nxt/
```

# Reference
https://nxt.engineering/blog/kafka-docker-image/


## Build schema registery image
1. Get the Kafka images repositiory
```shell

## Set up the subtree initially.
git subtree add --prefix schema-registry-images https://github.com/confluentinc/schema-registry-images tags/v7.0.2 --squash

## Pull newer versions
git subtree pull --prefix schema-registry-images https://github.com/confluentinc/schema-registry-images tags/v7.0.2 --squash
  
```

2. Add the confluent repository to the `schema-registry-images/pom.xml` file

```xml
<repositories>
    <repository>
        <id>confluent</id>
        <url>https://packages.confluent.io/maven/</url>
    </repository>
</repositories>
```

3. Build the schema-registry-images
```shell
cd schema-registry-images
mvn clean package \
  -DskipTests -Pdocker \
  -DCONFLUENT_PACKAGES_REPO='https://packages.confluent.io/rpm/7.0' \
  -DCONFLUENT_VERSION=7.0.2 \
  -Ddocker.registry=nxt/
```

## Verify the new Images
```shell

docker images | grep confluentinc

# Example output
nxt/confluentinc/cp-server-connect        7.0.1-ubi8               ccee5c62b5e7   2 minutes ago       2.13GB
nxt/confluentinc/cp-server-connect-base   7.0.1-ubi8               b8f607a05437   2 minutes ago       2.13GB
nxt/confluentinc/cp-kafka-connect         7.0.1-ubi8               a639af003ccc   3 minutes ago       1.42GB
nxt/confluentinc/cp-schema-registry       7.0.1-ubi8               d4ee053964d5   4 minutes ago       1.68GB
nxt/confluentinc/cp-kafka-connect-base    7.0.1-ubi8               596428b5c5c1   4 minutes ago       1.42GB
nxt/confluentinc/cp-enterprise-kafka      7.0.1-ubi8               9cbafe6c63ca   5 minutes ago       957MB
nxt/confluentinc/cp-kafka                 7.0.1-ubi8               d86c6260fd28   6 minutes ago       816MB
nxt/confluentinc/cp-server                7.0.1-ubi8               e5fca4aba934   7 minutes ago       1.58GB
nxt/confluentinc/cp-zookeeper             7.0.1-ubi8               b6bbc042ea65   8 minutes ago       816MB
nxt/confluentinc/cp-jmxterm               7.0.1-ubi8               6e3350e640a1   13 minutes ago      716MB
nxt/confluentinc/cp-base-new              7.0.1-ubi8               7e2fb9b02b52   14 minutes ago      708MB

```

