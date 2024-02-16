# NGINX
Docker Compose file that defines an Nginx OSS container and a MongoDB container connected to a custom network named "rondustech":

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:latest
    container_name: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    networks:
      - rondustech

  mongodb:
    image: mongo:latest
    container_name: mongodb
    networks:
      - rondustech

networks:
  rondustech:
    driver: bridge
```

In this Docker Compose file:

- We define two services: `nginx` and `mongodb`.
- For the `nginx` service:
  - We use the official Nginx OSS image (`nginx:latest`).
  - We map port 80 of the host to port 80 of the container to allow access to the Nginx server.
  - We mount a local `nginx.conf` file into the container at `/etc/nginx/nginx.conf` in read-only mode. You need to provide the `nginx.conf` file with your Nginx configuration.
  - We connect the `nginx` service to the `rondustech` network.

- For the `mongodb` service:
  - We use the official MongoDB image (`mongo:latest`).
  - We connect the `mongodb` service to the `rondustech` network.

- We define a custom network named `rondustech` with the bridge driver.

Make sure you have the `nginx.conf` file in the same directory as the Docker Compose file, and adjust it according to your Nginx configuration requirements.

To use this Docker Compose file, save it to a file named `docker-compose.yml` and then run:

```bash
docker-compose up -d
```

This will start both the Nginx and MongoDB containers connected to the `rondustech` network.
