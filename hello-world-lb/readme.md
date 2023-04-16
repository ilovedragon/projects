**Task**: Create a Docker Compose file that contains two containers that display "hello-world 1" and "hello-world 2", respectively. Add a load balancer to ensure that when we access "http://localhost", the traffic is distributed between the two containers in a round-robin fashion, so that the first request is sent to "hello-world1", the second to "hello-world 2", the third to "hello-world 1" again, and so on.

# **Overview**

This repository contains a Docker Compose configuration file that deploys a custom Flask application in two backend instances, and an Nginx load balancer to distribute incoming traffic between these two backend instances. The load balancer is set up to route incoming requests to one of the backend instances in a round-robin manner.

# **Prerequisites**
1. Before you begin, you will need to have Docker and Docker Compose installed on your system.
2. You will also need to create a Flask app image called myapp

**How to create a Flask app image called myapp:**

1. Clone a docker examples repo by running: git clone https://github.com/docker/awesome-compose.git. 
2. cd awesome-compose/flask
3. Inside the flask directory, run: docker build -t myapp app/. 
4. docker images to list myapp image is created.

# **Create hello-world-lb container:**
1. Copy or gitclone the files to your project directory.
2. app1.py and app2.py will display hello world 1 or 2 in container based on myapp image.
3. nginx.conf configure the load balancer
4. docker compose up -d
5. This will start all the services and the load balancer will be accessible at http://localhost. When you access http://localhost/, you should see the "Hello World 1!" message, and when you refresh http://localhost/ again, you should see the "Hello World 2!" message.!"

## **Code Explanation**
**Docker Compose**
The docker-compose.yml file defines the services that will be used to run the application, including:

1. The lb service, which uses the Nginx image as the load balancer and maps port 80 on the host to port 80 in the container. The nginx.conf file is mounted as a volume to configure the load balancer.
2. The app1 and app2 services, which use the myapp image and map port 8001 and 8002 on the host to port 8000 in the container, respectively. The app1.py and app2.py files are used to create a custom message in the container of Flask server.
3. Volumes allow the containers to share data with the host or with other containers, and that in this particular Docker Compose file, volumes are used to:

    - Mount the "nginx.conf" file onto the "lb" container, which is running the Nginx image as a load balancer. This allows us to customize the configuration of the load balancer without having to modify the image itself.

    - Mount the "app1.py" and "app2.py" files onto the "app1" and "app2" containers, respectively, which are running the "myapp" image with a Flask application. This allows us to customize the message in the container of Flask server, without having to rebuild the image each time we make a change to the code.

docker-compose.yml
```
version: '3.9'

services:
  lb:
    image: nginx
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - app1
      - app2
  app1:
    image: myapp
    ports:
      - "8001:8000"
    volumes:
      - ./app1.py:/app/app.py
  app2:
    image: myapp
    ports:
      - "8002:8000"
    volumes:
      - ./app2.py:/app/app.py
```

**Flask Application**
The myapp Flask application returns a custom message.  By default, the app listens on port 8000 for incoming requests using the "app.py" file, and it will return the message "Hello, World!":

app.py
```
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)
``` 

However, you can customize the text by creating two additional files: "app1.py" and "app2.py". "app1.py" will return the message "Hello, World 1!", while "app2.py" will return the message "Hello, World 2!".

app1.py:
```
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello, World 1!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)
``` 

app2.py:
```
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello, World 2!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)
``` 

**Nginx Configuration**
The nginx.conf file is a configuration file for Nginx, we are using Nginx here as both a reverse proxy and a load balancer.

The first line sets the number of worker processes to 1, and the events block specifies that each worker process can handle up to 1024 connections.

The http block contains the main configuration for the HTTP server. In this configuration, an upstream block is defined, which specifies a list of backends that Nginx can use to distribute traffic. In this case, the upstream block specifies two servers: app1:8000 and app2:8000. The configuration uses a simple round-robin algorithm to distribute traffic evenly between the services.

The server block defines a virtual server that listens on port 80. The location / block is used to handle requests to the root URL. The proxy_pass directive tells Nginx to forward requests to the backend upstream, which will distribute the traffic to the two specified servers.

The proxy_set_header directives are used to set HTTP headers in the forwarded request.:

nginx.conf
```
worker_processes 1;

events { worker_connections 1024; }

http {
  upstream backend {
    server app1:8000;
    server app2:8000;
  }

  server {
    listen 80;

    location / {
      proxy_pass http://backend;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded
```


