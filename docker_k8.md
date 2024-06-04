# General Notes for Docker, DockerSwarm and Kubernetes(k8 latter on)

## Installation

- [Docker Engine Installation on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)

### Run Docker as Non-Root User

- [Managing Docker as a Non-Root User](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user)

## Nginx with self signed Certificates

1. Create a directory for the website:

    ```bash
    mkdir website
    ```

2. Create and edit the index.html file:

    ```bash
    nano website/index.html
    ```

3. Edit the nginx.conf file:

    ```bash
    nano nginx.conf
    ```

    ```nginx
    worker_processes 1;

    events {
        worker_connections 1024;
    }

    http {
        server {
            listen 443 ssl;
            server_name 54.227.155.225;

            ssl_certificate /etc/nginx/ssl/nginx.crt;
            ssl_certificate_key /etc/nginx/ssl/nginx.key;

            location / {
                root /usr/share/nginx/html;
                index index.html;
            }
        }
    }
    ```

4. Create a Dockerfile:

    ```Dockerfile
    FROM nginx:latest

    # Copy from local machine to container
    COPY index.html /usr/share/nginx/html/
    COPY nginx.conf /etc/nginx/nginx.conf
    COPY ssl /etc/nginx/ssl

    EXPOSE 443
    ```

5. Generate SSL certificates:

    ```bash
    openssl req -x509 -nodes -newkey rsa:2048 -keyout ssl/nginx.key -out ssl/nginx.crt
    ```

6. Build the Docker image:

    ```bash
    docker build -t nginx-webiste .
    ```

7. Run the Docker container:

    ```bash
    docker run -d -p 443:5000 --name nginx-webiste-container nginx-webiste
    ```

## Flask Project

- [Docker Compose Getting Started](https://docs.docker.com/compose/gettingstarted/#step-1-set-up)

## Remove All Docker Resources

```bash
docker system df -v
docker system prune -a
docker image prune
docker container prune
docker volume prune
docker system df -v
```

## Docker publish to docker hub

- https://www.geeksforgeeks.org/docker-publishing-images-to-docker-hub/

## Add nodes to dockerswarm

- https://docs.docker.com/engine/swarm/swarm-tutorial/create-swarm/
- 

## Build docker service(swarm)

```bash
docker build -t username/repo-name .
docker push username/repo-name
docker service create --name <name-you-want> --replicas <x> -p x:x username/repo-name
```

## Rebuild docker service(swarm)

```bash
docker build -t username/repo-name .
docker push username/repo-name
docker service update --image username/repo-name service-name
```


## Useful commands

- sudo python3 -m pip install -r requirements.txt
