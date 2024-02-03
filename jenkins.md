# Jenkins
### Below is a simple example of a Dockerfile for building a Jenkins image:

**Dockerfile:**
```Dockerfile
### Use the official Jenkins LTS base image
FROM jenkins/jenkins:lts

### Skip the initial setup wizard
ENV JAVA_OPTS -Djenkins.install.runSetupWizard=false

### Install necessary plugins (you can customize this list)
RUN /usr/local/bin/install-plugins.sh \
    docker \
    pipeline

### Copy Jenkins configurations
COPY config/*.xml /var/jenkins_home/

### Expose ports for Jenkins web interface and the JNLP agent
EXPOSE 8080 50000

### Use "jenkins" as the default user
USER jenkins
```

This Dockerfile uses the official Jenkins LTS base image, installs some plugins, and skips the initial setup wizard. The configurations are copied from the `config` directory, which you can customize based on your needs.

**Command to Build and Run the Image:**
```bash
docker build -t my-jenkins-image .
docker run -p 8080:8080 -p 50000:50000 --name my-jenkins-container my-jenkins-image
```

Explanation:
- `docker build -t my-jenkins-image .`: This command builds the Docker image using the Dockerfile in the current directory (`.`) and tags it as `my-jenkins-image`.
- `docker run -p 8080:8080 -p 50000:50000 --name my-jenkins-container my-jenkins-image`: This command runs a container from the built image. It maps host ports 8080 and 50000 to container ports 8080 and 50000, respectively. The `--name` option gives the container the name `my-jenkins-container`.

After running the above commands, you should be able to access Jenkins at http://localhost:8080 in your web browser. During the initial setup, you can unlock Jenkins using the initial admin password displayed in the console logs.

Note: Ensure that you have Docker installed and running on your machine before executing these commands.
