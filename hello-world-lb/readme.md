**Task**: Create a docker compose file to have two hello-world container, one will display hello-world1 another will show hello-world2. Add another load-balancer, so that whenever we point to http://localhost, first will display hello-world1, another will show hellp-world2

# **Overview**

The code in this repository uses Docker Compose to deploy two instances of a custom Flask application, each with a unique message, and an Nginx load balancer to route traffic between them. The load balancer is configured to return a custom response based on the URI path of the request, displaying the appropriate message for the app instance.

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
5. This will start all the services and the load balancer will be accessible at http://localhost. When you access http://localhost/app1, you should see the "Hello World 1!" message, and when you access http://localhost/app2, you should see the "Hello World 2!" message.!"

## **Code Explanation**
**Docker Compose**
The docker-compose.yml file defines the services that will be used to run the application, including:

1. The lb service, which uses the Nginx image as the load balancer and maps port 80 on the host to port 80 in the container. The nginx.conf file is mounted as a volume to configure the load balancer.
3. The app1 and app2 services, which use the myapp image and map port 8001 and 8002 on the host to port 8000 in the container, respectively. The app1.py and app2.py files are used to create a custom message in the container of Flask server.

**Flask Application**
The myapp Flask application consists of a single route that returns a custom message. The default app.py file defines the Flask app and listens on port 8000 for incoming requests:

```
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8000)
``` 

app1.py, app2.py replace this default app.py files, to customize the text.

**Nginx Configuration**
The nginx.conf file is a configuration file for Nginx, a popular web server and reverse proxy. The first line sets the number of worker processes to 1, and the events block specifies that each worker process can handle up to 1024 connections.

The http block contains the main configuration for the HTTP server. In this configuration, an upstream block is defined, which specifies a list of backends that Nginx can use to distribute traffic. In this case, the upstream block specifies two servers: app1:8000 and app2:8000. The configuration uses a simple round-robin algorithm to distribute traffic evenly between the services.

The server block defines a virtual server that listens on port 80. The location / block is used to handle requests to the root URL. The proxy_pass directive tells Nginx to forward requests to the backend upstream, which will distribute the traffic to the two specified servers.

The proxy_set_header directives are used to set HTTP headers in the forwarded request.:


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


