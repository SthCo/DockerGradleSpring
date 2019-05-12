# README

## Requirements
* Docker

## Install Docker on Linux

Follow [the following link](https://docs.docker.com/install/linux/docker-ee/ubuntu/)

## Init

* ```docker run --name=sr03-project-db --env="MYSQL_ROOT_PASSWORD=root" --env="MYSQL_PASSWORD=root" --env="MYSQL_DATABASE=sr03-project-db" mysql ```

* Create a Dockerfile within the [following folder](https://github.com/itsromiljain/gradle-springboot) with the following code inside :
___
```
FROM java:8

VOLUME /tmp

EXPOSE 8080

ADD /build/libs/gradle-springboot-1.0.jar gradle-springboot-1.0.jar

ENTRYPOINT ["java","-jar","gradle-springboot-1.0.jar"]
```
___

* ```docker build -t sr03-project .```

* ```docker run -t --name sr03-project --link sr03-project-db:mysql -p 8080:8080 sr03-project```

* ```docker exec -it sr03-project bash```

You should obtain something like this in the bash file :
___
 “172.17.0.2 mysql 93c261be3855 docker-mysql”
___
* Replace the build.gradle with :
___
```
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath("org.springframework.boot:spring-boot-gradle-plugin:1.5.8.RELEASE")
        classpath('se.transmode.gradle:gradle-docker:1.2')
    }
}

apply plugin: 'java'
apply plugin: 'eclipse'
apply plugin: 'org.springframework.boot'
apply plugin: 'docker'

description = "gradle-springboot"

jar {
    baseName = 'gradle-springboot'
    version =  '1.0'
}

def javaVersion = JavaVersion.VERSION_1_8

sourceCompatibility = javaVersion
targetCompatibility = javaVersion

tasks.withType(JavaCompile) {
	options.encoding = 'UTF-8'
}

repositories {   
     mavenCentral()
}

dependencies {
    compile group: 'org.springframework.boot', name: 'spring-boot-starter-web', version:'1.5.8.RELEASE'
    compile group: 'org.springframework.boot', name: 'spring-boot-starter-tomcat', version: '1.5.8.RELEASE'
    compile group: 'org.springframework.boot', name: 'spring-boot-starter-data-jpa', version:'1.5.8.RELEASE'
    compile group: 'mysql', name: 'mysql-connector-java', version:'5.1.44'
    compile group: 'commons-dbcp', name: 'commons-dbcp', version:'1.4'
    compile group: 'org.apache.commons', name: 'commons-collections4', version:'4.1'
    compile group: 'org.springframework.boot', name: 'spring-boot-legacy', version:'1.1.0.RELEASE'
}

task buildDocker (type:Docker, dependsOn: build) {
	applicationName = jar.baseName
	dockerfile = file('Dockerfile')
	doFirst {
		copy {
			from jar
			into stageDir
		}
	}
}
```
___

* ```gradle build buildDocker```

* Create a file named "docker-compose.yml" at the root of
your project with the following content :

___
```
version: '3'

services:
  docker-mysql:
    image: mysql:latest
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=test
      - MYSQL_PASSWORD=root
  spring-boot-jpa-docker-webapp:
    image: gradle-springboot
    depends_on:
      - docker-mysql
    ports:
      - 8080:8080
    environment:
      - DATABASE_HOST=sr03-project
      - DATABASE_USER=root
      - DATABASE_PASSWORD=root
      - DATABASE_NAME=sr03-project-db
      - DATABASE_PORT=3306

```
___

* ```docker-compose up```

* If you want to go into docker folder on Mac :
  * ```screen ~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/tty```
  * ```cd /var/lib/docker```
