# Tomcat
### Below is a simple Dockerfile to build a Tomcat image. And commands to build and run the Docker image.

**Dockerfile:**

```Dockerfile
### Use the official Tomcat base image from Docker Hub
FROM tomcat:latest

### Copy your web application (WAR file) to the Tomcat webapps directory
COPY ./your-web-app.war /usr/local/tomcat/webapps/

### Optionally, you can expose the Tomcat port (8080) if needed
EXPOSE 8080
```

Replace `./your-web-app.war` with the path to your actual web application WAR file.

**Build and Run Commands:**

Assuming your Dockerfile is in the same directory as your web application WAR file, you can use the following commands:

1. **Build the Docker image:**

    ```bash
    docker build -t your-tomcat-image-name .
    ```

   Replace `your-tomcat-image-name` with the desired name for your Tomcat image.

2. **Run the Docker container:**

    ```bash
    docker run -p 8080:8080 --name your-tomcat-container-name your-tomcat-image-name
    ```

   Replace `your-tomcat-container-name` with the desired name for your Tomcat container.

   The `-p 8080:8080` flag maps port 8080 on the host to port 8080 on the container, allowing you to access the Tomcat server.

Now, your Tomcat container should be running, and you can access your web application by navigating to `http://localhost:8080/your-web-app`.
