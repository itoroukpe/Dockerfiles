
## A multi-stage Dockerfile for Maven, Jenkins, SonarQube, Nexus, and Tomcat 

```Dockerfile
# Stage 1: Build Maven project
FROM maven:3.8.4-openjdk-11 AS build
WORKDIR /app
COPY . /app
RUN mvn clean install

### Stage 2: Build Jenkins image
FROM jenkins/jenkins:lts AS jenkins
USER root
RUN apt-get update && apt-get install -y maven
USER jenkins
COPY --from=build /app/target/*.war /usr/share/jenkins/ref/war/WEB-INF/lib/

### Stage 3: Build SonarQube image
FROM sonarqube:8.9.0-community
COPY --chown=sonarqube:sonarqube --from=build /app/target /opt/sonarqube/extensions/plugins/

### Configure SonarQube user
RUN echo "sonar.security.realm=LDAP" >> /opt/sonarqube/conf/sonar.properties
RUN echo "sonar.security.savePassword=true" >> /opt/sonarqu e/conf/sonar.properties
RUN echo "sonar.security.updateUserAttributes=true" >> /opt/sonarqube/conf/sonar.properties
RUN echo "sonar.security.localUsers=admin" >> /opt/sonarqube/conf/sonar.properties

### Stage 4: Build Nexus image
FROM sonatype/nexus3:3.34.1
COPY --chown=nexus:nexus --from=build /app/target/*.war /nexus-data/nexus3/plugin-repository/

### Stage 5: Build Tomcat image
FROM tomcat:9.0.54-jdk11-openjdk
COPY --from=build /app/target/*.war /usr/local/tomcat/webapps/

### Expose ports for the applications
EXPOSE 8080 8081 9000 9092 9093

### Start Tomcat
CMD ["catalina.sh", "run"]
```

#### Please note:

- The Maven project is built in the first stage.
- Jenkins, SonarQube, Nexus, and Tomcat images are built in separate stages using their respective base images.
- The Jenkins image is configured to include the built WAR file.
- The SonarQube image is configured to include the Maven plugin and properties.
- Nexus is configured to include the built WAR file.
- Tomcat is configured to include the built WAR file.

Ensure you replace any version numbers in the Dockerfile with the latest versions available for each component. Also, this example assumes that the WAR file produced by the Maven project is the target artifact you want to deploy.

Remember to customize the Dockerfile based on your specific requirements and configurations.
