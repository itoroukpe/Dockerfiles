## Sonarqube
### Below is a basic example of a Dockerfile for building a SonarQube image. This example assumes you are using the Community Edition of SonarQube.

```dockerfile
# Use the official OpenJDK base image
FROM openjdk:11-jre-slim

### Set environment variables
ENV SONARQUBE_VERSION=9.1 \
    SONARQUBE_HOME=/opt/sonarqube

### Download and install SonarQube
RUN set -eux; \
    groupadd -r sonarqube && useradd -r -g sonarqube -d $SONARQUBE_HOME sonarqube; \
    apt-get update; \
    apt-get install -y --no-install-recommends wget unzip; \
    wget -O /tmp/sonarqube.zip "https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-$SONARQUBE_VERSION.zip"; \
    unzip /tmp/sonarqube.zip -d /opt; \
    mv "/opt/sonarqube-$SONARQUBE_VERSION" $SONARQUBE_HOME; \
    chown -R sonarqube:sonarqube $SONARQUBE_HOME; \
    rm -rf /tmp/sonarqube.zip; \
    apt-get purge -y --auto-remove wget unzip; \
    rm -rf /var/lib/apt/lists/*

### Expose the default SonarQube port
EXPOSE 9000

# Set user to run SonarQube
USER sonarqube

### Start SonarQube
CMD ["sh", "-c", "$SONARQUBE_HOME/bin/linux-x86-64/sonar.sh"]
```
Sure, here's a basic Dockerfile for building a SonarQube image:


### This Dockerfile uses the official SonarQube image as a base and exposes the default SonarQube web port (9000).

### To build and run the Docker image, follow these steps:

1. Save the Dockerfile in a directory (e.g., `sonarqube-docker`).

2. Open a terminal and navigate to the directory containing the Dockerfile.

3. Build the Docker image using the following command:

   ```bash
   docker build -t my-sonarqube-image .
   ```

   Replace `my-sonarqube-image` with your desired image name.

4. Once the image is built, you can run a container from it using the following command:

   ```bash
   docker run -d -p 9000:9000 --name sonarqube-container my-sonarqube-image
   ```

   - `-d`: Run the container in the background (detached mode).
   - `-p 9000:9000`: Map the host machine's port 9000 to the container's port 9000.
   - `--name sonarqube-container`: Assign a name to the container (you can choose any name).
   - `my-sonarqube-image`: The name of the Docker image you built.

Now, you should have a SonarQube container running, and you can access the SonarQube web interface by navigating to `http://localhost:9000` in your web browser.


### This Dockerfile does the following:

1. Uses the official OpenJDK 11 base image.
2. Sets environment variables for SonarQube version and installation path.
3. Creates a `sonarqube` user and group.
4. Downloads and installs SonarQube.
5. Exposes the default SonarQube port 9000.
6. Sets the user to run SonarQube.
7. Defines the command to start SonarQube.

Note: Make sure to check the [official SonarQube download page](https://www.sonarqube.org/downloads/) for the latest version and adjust the `SONARQUBE_VERSION` accordingly in the Dockerfile.
