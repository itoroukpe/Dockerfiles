
## A multi-stage Dockerfile for Maven, Jenkins, SonarQube, Nexus, and Tomcat 

```Dockerfile
# Stage 1: Build Maven project
FROM maven:3.8.4-openjdk-11 AS build
WORKDIR /app
COPY . /app
RUN mvn clean install

# Stage 2: Build Jenkins image
FROM jenkins/jenkins:lts AS jenkins
USER root
RUN apt-get update && apt-get install -y maven
USER jenkins
COPY --from=build /app/target/*.war /usr/share/jenkins/ref/war/WEB-INF/lib/

# Stage 3: Build SonarQube image
FROM sonarqube:8.9.0-community
COPY --chown=sonarqube:sonarqube --from=build /app/target /opt/sonarqube/extensions/plugins/

# Configure SonarQube user
RUN echo "sonar.security.realm=LDAP" >> /opt/sonarqube/conf/sonar.properties
RUN echo "sonar.security.savePassword=true" >> /opt/sonarqu e/conf/sonar.properties
RUN echo "sonar.security.updateUserAttributes=true" >> /opt/sonarqube/conf/sonar.properties
RUN echo "sonar.security.localUsers=admin" >> /opt/sonarqube/conf/sonar.properties

# Stage 4: Build Nexus image
FROM sonatype/nexus3:3.34.1
COPY --chown=nexus:nexus --from=build /app/target/*.war /nexus-data/nexus3/plugin-repository/

# Stage 5: Build Tomcat image
FROM tomcat:9.0.54-jdk11-openjdk
COPY --from=build /app/target/*.war /usr/local/tomcat/webapps/

# Expose ports for the applications
EXPOSE 8080 8081 9000 9092 9093

# Start Tomcat
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

========================================================

Creating a multi-stage Dockerfile for a setup involving Git, Maven, Jenkins, SonarQube, Nexus, and Tomcat is a complex task. Below is a simplified example to give you an idea. This example assumes that the required configurations and setup steps for each tool are handled appropriately in your actual deployment.

```Dockerfile
# Stage 1: Build stage with Maven, Git, and Jenkins
FROM maven:3.8.4-openjdk-17-slim AS builder

WORKDIR /app

# Clone the Git repository
RUN apt-get update && \
    apt-get install -y git && \
    git clone https://github.com/itoroukpe/maven-web-application.git .

# Build the Maven project
RUN mvn clean install

# Stage 2: SonarQube Scanner stage
FROM sonarsource/sonar-scanner-cli:latest AS sonarqube-scanner

WORKDIR /app

COPY --from=builder /app /app

# Run SonarQube scanner
RUN sonar-scanner

# Stage 3: Nexus stage
FROM sonatype/nexus3:latest AS nexus

# Additional Nexus configuration if needed

# Stage 4: Tomcat stage
FROM tomcat:9.0-jdk17-openjdk-slim AS tomcat

# Additional Tomcat configuration if needed

# Stage 5: Final stage with only necessary artifacts
FROM openjdk:17-slim

# Copy artifacts from previous stages
COPY --from=builder /app/target/*.war /usr/local/tomcat/webapps/

COPY --from=sonarqube-scanner /app /app

COPY --from=nexus /nexus-data /nexus-data

COPY --from=tomcat /usr/local/tomcat /usr/local/tomcat

# Additional configuration if needed

# Expose ports
EXPOSE 8080 8081 9000

CMD ["catalina.sh", "run"]
```

This example uses multi-stage builds to separate the build environment and the final production environment. You would need to customize each stage to match the specific configurations and requirements of your tools and applications.

Note: This is a simplified example, and depending on your specific needs, you might need to adapt and enhance the Dockerfile to meet your deployment requirements. Always ensure you follow best practices for securing and configuring each tool in a production environment.
