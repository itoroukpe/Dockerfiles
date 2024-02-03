# Nexus
## This Dockerfile uses the official Nexus Repository Manager image as the base image and adds some custom configurations:

```Dockerfile
### Use the official Nexus Repository Manager base image
FROM sonatype/nexus3:latest

### Author
LABEL maintainer="Your Name <your.email@example.com>"

### Set environment variables for Nexus
ENV NEXUS_VERSION=3.0.0 \
    NEXUS_HOME=/opt/sonatype/nexus \
    NEXUS_DATA=/nexus-data \
    SONATYPE_WORK=/opt/sonatype/sonatype-work

### Custom configurations (Optional)
COPY nexus-oss-feature-bundle-template-3.x.milestone7.jar ${NEXUS_HOME}/system/org/sonatype/nexus/assemblies/nexus-oss-feature-bundle/3.x.milestone7/nexus-oss-feature-bundle-3.x.milestone7.jar

### Expose Nexus ports
EXPOSE 8081

### Start Nexus
CMD ["sh", "-c", "${NEXUS_HOME}/bin/nexus run"]
```

### This Dockerfile does the following:

1. Uses the official Nexus Repository Manager base image (`sonatype/nexus3:latest`).
2. Sets some environment variables related to Nexus.
3. Optionally adds custom configurations (e.g., adding a Nexus feature bundle jar).
4. Exposes the default Nexus port (8081).
5. Defines the command to start Nexus.

You can customize this Dockerfile based on your specific requirements. To build the Docker image, you can use the following command in the directory where the Dockerfile is located:

```bash
docker build -t your-nexus-image:tag .
```

Replace `your-nexus-image:tag` with the desired image name and tag.

After building the image, you can run a Nexus container using:

```bash
docker run -d -p 8081:8081 --name nexus-container your-nexus-image:tag
```

This assumes you want to map the container's port 8081 to the host's port 8081. Adjust the port mapping according to your needs.
