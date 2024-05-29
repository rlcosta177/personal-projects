Installation:
  - https://docs.docker.com/engine/install/ubuntu/

Run docker as non root user:
  - https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user


Nginx:
  - mkdir website
  - nano website/index.html (put something in the index.html)
  - nano nginx.conf
    
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

  - nano Dockerfile
    
      
    
      FROM nginx:latest

      # cp from local machine to container
      COPY index.html /usr/share/nginx/html/
      COPY nginx.conf /etc/nginx/nginx.conf
      COPY ssl /etc/nginx/ssl

      EXPOSE 443
    
      

  - openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout ssl/nginx.key -out ssl/nginx.crt
  - docker run -d -p 443:443 --name my-https-container my-https-site
  - docker build -t my-https-site .



flask project:
  - https://docs.docker.com/compose/gettingstarted/#step-1-set-up



remove all:
  - docker system df -v
  - docker system prune -a
  - docker image prune
  - docker container prune
  - docker volume prune
  - docker system df -v


publish docker container to docker hub:
  - https://www.geeksforgeeks.org/docker-publishing-images-to-docker-hub/
