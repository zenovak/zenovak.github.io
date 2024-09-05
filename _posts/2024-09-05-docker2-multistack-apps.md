---
title: Docker - 2 Dockerise Multistack Applications
date: 2024-09-05 #TTTT Means timezone
categories: [Docker, DockerI-Basics]
tags: [docker, nextjs, flask, python]     # TAG names should always be lowercase
---

## The problem
Let's say we have a nextjs frontend and a python backend used to process some data. The current approach of deploying this project:

On the nextjs stack
```
npm install
npm run build
npm run start
```

On the python flask stack:
```
pip install -r requiremnets.txt
python3 app.py 
```

This is complicated and also requires that you configure dependencies like gunicorn, etc.

<br>

With docker, we can one-click deploy this application. So here's what we need to configure and setup:
- We need to setup a CORS configuration so these 2 stacks can talk to each other without docker to clear configuration confusions.
- We need to containerize both nextjs and python stack
- We need to write a docker file to build this docker image

In docker a common approach to build our application is back to front. That means we should start with 
- dependencies of the backend (Postgres image?)
- backend stack, flask
- frontend stack, nextjs

<br><br>

---
## Python stack 
In your python stack, you need to configure a backend API with CORS. You can use the flask-cors package.

```python
from flask import Flask  
from flask_cors import CORS  
  
  
app = Flask(__name__)  
CORS(app)  

  
@app.route("/api/hello_world")  
def hello_world():  
    return {  
        "message": "Hello, World"  
    }  
  
  
if __name__ == "__main__":  
    app.run(debug=True)
```



### Dockerfile
The `Dockerfile` for this application. You should name it as `flask.dockerfile` as we will be using 1 dockerfile for each stack, it helps with clearer seperation of concern.:
```dockerfile
FROM --platform=$BUILDPLATFORM python:3.11-alpine AS builder

WORKDIR /app

COPY requirements.txt ./

RUN pip install -r "requirements.txt"
  
COPY . .

EXPOSE 4000

CMD ["flask", "run", "--host=0.0.0.0", "--port=4000"]
```

So currently your project's structure from the root should look like this:

```
root:
    backend/
        - venv/
        - app.py
        - flask.dockerfile
        
    fronend/

```


### Compose.yml
Next. we'll create a docker compose file to test our dockerfile, its image and deployment. create a `compose.yml` at the root of the repository.
```yml
services:
    
    # flask service
    flask:
    container_name: flask
    image: flask:1.0.0
    build:
        context: ./backend
        dockerfile: flask.dockerfile
    ports:
        - 4000:4000
```

we can run this docker application via the docker commands:
```
docker compose build flask
docker compose up
```

<br><br>

---
## Nextjs stack
First thing to do is configure the proxy, so your backend can be forwarded without a CORS error making a fuss.

Its important to ensure on `fetch()` requests are made using a relative path rather than the docker's expose port path.

Example, your python backend has an endpoint on `localhost:4000/api/hello`. You should make your fetch request in the nextjs frontend as:
```js
// correct
fetch("/api/hello");

// wrong
fetch("http://localhost:4000/api/hello")
```

Then forward the requests by configuring a proxy in `next.config.mjs`
```js
const nextConfig = {
    // other configs...
    async rewrites() {
        return [
            {
                source: '/api/:path*',
                // Proxy to python backend. flask is service name from compose.yml
                destination: 'http://flask:4000/api/:path*', 
            },
        ]
      },
}

// ...
```

We need to use the relative path because, when we deploy on a VPS and a user makes a request. `http://localhost:4000/api/hello` becomes a hard coded value.

Our python backend doesnt sit on `localhost` for the user. it sits on our server, with a URL IP address. Thus its better to allow node to infer the api endpoint destination instead.

<br>

### Dockerize
The docker file configuration is quite complicated, and thus its recommended to simply follow the template instructions from the official docs [here](https://github.com/vercel/next.js/blob/canary/examples/with-docker/Dockerfile)

You need to copy the docker file, and configure the `next.config.mjs`:
```js
// next.config.js
module.exports = {
  // ... rest of the configuration.
  output: "standalone",
};
```

And then your `compose.yml`
```yml
services:
    # nextjs frontend service
    next:
    container_name: next
    image: next:1.0.0
    build:
        context: ./frontend
        dockerfile: next.dockerfile
    ports:
        - 3000:3000
    depends_on:
        - flask
    
    # flask service
    flask:
        container_name: flask
        image: flask:1.0.0
        build:
            context: ./backend
            dockerfile: flask.dockerfile
        ports:
            - 4000:4000
```

once this is setup we can run our application

```
docker compose up -d next
```

<br><br>

---
## Networking pains and gotchas
> [stackoverflow](https://stackoverflow.com/questions/74854996/next-js-fetch-get-an-econnrefused-error-in-docker-strapi-as-backend)

Let's imagine we are running this docker application on our local computer. We can access our 2 stacks via the browser:

- http://localhost:4000/api/hello_world our flask backend
- http://localhost:3000/ our nextjs frontend

However. When we make a http request from the frontend to the backend. There are 2 main ways this goes down.

- The request is made in the client browser. `fetch(http://localhost:4000/api/hello_world)`. Where fetch here is made via a hardcoded URL. 

It works on our local computer, but not on a VPS. To make it work on the VPS we need to make the base URL as an .env variable, and change it to the public facing address of our backend.

<br>

- We use the `next.config.mjs` to reroute. The request is made in the client browser as `fetch(/api/hello_world)`,  nodejs translate this address at runtime **server side**.

we use the **docker service name**  `destination: 'http://flask:4000/api/:path*` because, nextjs application sits on in its own container. Thus, its `localhost:4000` is non existent. To talk to the flask container, it must use the flask service name and comunicate via the docker network. 

However when we run flask and nextjs as standalone applications, its sharing a single environment, thus serverside, `localhost:4000` exists

```
On a local computer, the proxy works, by forwarding the requests to localhost:400
                                         Local computer
                                       _____________________________
    http://localhost:3000/api/hello   |     proxy:localhost:4000    |
Dev  -------------------------------->|  nextjs <----------> flask  |
                                      |_____________________________|
frontend coded as
fetch(/api/hello)


But also asny requests would work on the local computer, as both processes are 
running on localhost
                                         Local computer
                                       _____________________________
    http://localhost:4000/api/hello   |   flask                     |
Dev  -------------------------------->|   nextjs                    |
     http://localhost:3000/           |_____________________________| 

frontend coded as
fetch(http://localhost:4000/api/hello)


But for the user. The processes are not running on their localhost, but the
server's. Thus we must proxy via next.config
                                         Server
                                   _____________________________
    http://site.com/api/hello     |     proxy:localhost:4000    |
User ---------------------------> |  nextjs <----------> flask  |
                                  |_____________________________|
frontend coded as
fetch(/api/hello)


But when we containerize our application. The environments are separated, 
in NextJS's container, no processes are running localhost:400,
thus it must be rerouted via docker's. 
The server cannot access flask:4000, as it is not part of the docker network.
                                        Server w/ docker
                                   _____________________________________
    http://site.com/api/hello     |          proxy:flask:4000           |   
User ---------------------------> |   _______       |        _______    |
                                  |  |nextjs | <==========> | flask |   |
frontend coded as                 |  |_______|    docker    |_______|   |
fetch(/api/hello)                 |      |        Network       |       |
                                  |      |                      |       |
                                  |  container              container   |
                                  |_____________________________________|


If we use a direct URL from the user to access the flask backend, 
an Nginx proxy will have to be configured.
                                        Server w/ docker
                                   _____________________________________
    http://site.com/api/hello     |     Nginx ----------------          |   
User ---------------------------> |   _______  proxy :api/*  _|_____    |
                                  |  |nextjs |              | flask |   |
frontend coded as                 |  |_______|              |_______|   |
fetch(http://site.com/api/hello)  |      |                      |       |
                                  |      |                      |       |
                                  |  container              container   |
                                  |_____________________________________|
         
```

<br>

### Share network namespaces instead
Instead of going thru so much headaches of configuring networks. A better way is to allow both of these services to share the server's localhost network

```yml
services:
    
    # flask service
    flask:
    container_name: flask
    image: flask:1.0.0
    build:
        context: ./backend
        dockerfile: flask.dockerfile
        network: host
    ports:
        - 4000:4000
```

NextJS can access the backend via `localhost:4000` on the server side no issues. For the client side making request from the browser. Its still best to configure the `next.config`.


<br><br>

---
## References

https://github.com/FrancescoXX/fullstack-flask-app/blob/main/frontend/next.dockerfile

https://github.com/docker/awesome-compose/tree/master/nginx-flask-mongo/flask

https://github.com/vercel/next.js/blob/canary/examples/with-docker/Dockerfile

https://www.youtube.com/watch?v=1afIORRyp58

https://stackoverflow.com/questions/74854996/next-js-fetch-get-an-econnrefused-error-in-docker-strapi-as-backend/78951586#78951586
