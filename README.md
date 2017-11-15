# Using Dockerfiles with ```docker-compose``` using relative paths

## Project Tree
```
.
|_____README.md
|_____package.json
|_____conf
|_____|_____docker
|_____|_____|_____Dockerfile
|_____|_____|_____Dockerfile.dev
|_____|_____|_____Dockerfile.prod
|_____|_____|_____docker-compose.dev.yml
|_____|_____|_____docker-compose.network.yml
|_____|_____|_____docker-compose.prod.yml
|_____src
|_____|_____server.js
```

*File Descriptions*

- __README.md__ - The document you are reading.
- __package.json__ - Defines the dependencies, entry point and startup script for the project.
- __conf/docker/Dockerfile__ - Builds a container for the project src.
- __conf/docker/Dockerfile.dev__ - Builds a container for the project src in ```dev mode```.
- __conf/docker/Dockerfile.prod__ - Builds a container for the project src in ```production mode```.
- __conf/docker/docker-compose.dev.yml__ - Orchestrates the instantiation of the containers used in the project in ```dev mode```.
- __conf/docker/docker-compose.prod.yml__ - Orchestrates the instantiation of the containers used in the project in ```production``` mode.
- __conf/docker/docker-compose.network.yml__ - Orchestrates the instantiation of the containers used in the project in ```network``` mode.
- __src/server.js__ - The actual project src code that executes in the container.


## Workflow Logic

The project is run with (one of) the following command from the root dir ( ```.``` in the tree ):
```
$ docker-compose -f conf/docker/docker-compose.dev.yml up

$ docker-compose -f conf/docker/docker-compose.prod.yml up

$ docker-compose -f conf/docker/docker-compose.network.yml up
```

1) The file docker-compose.yml defines the service(s), build context, dockerfile(s) to use, and any other configuration details salient to the specific container(s) running (including args, ports, labels, tags, names, etc.)

1) The Dockerfile specified by the service being containerized by the docker-compose.yml file will now build the container image (layer by layer - caching where possible) and deploy the container once it has succesfully been built and configured. These steps include (but ar enot limited too):
    - defining the base image to build the container from.
    - define directories used in the container/project.
    - copy src files into the container form the host system.
    - install dependencies into the container.
    - expose ports
    - issue runtime commands (eg. 'start')

*NOTE: All docker-compose actions are executed relative to the path location of the docker-compose.yml file or its specific build context value.*

*NOTE: All Dockerfile actions are executed relative to the path location of the Dockerfile file.*

*NOTE: You can supply multiple -f configuration files. When you supply multiple files, Compose combines them into a single configuration. Compose builds the configuration in the order you supply the files. Subsequent files override and add to their predecessors.*

*NOTE: You must provide a unique name for each image your docker-compose file generates. This is based on the service name used in the docker-compose.yml file. Two services of the same name cannot run on the same host, even if defined in different docker-compose files. They will generate image names that conflict in the image listing. Therefore best practice is to use unique names for all running services.*


## Project Source

__Package.json__

This file defines the metadata for the project app, specifies dependencies, and defines the scripts used by docker to run the application.

```
{
  "name": "docker-nodejs-webapp-example",
  "version": "1.0.0",
  "description": "Node.js in docker",
  "author": "",
  "license": "ISC",
  "main": "server.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node src/server.js"
  },
  "dependencies": {
    "express": "^4.13.3",
    "ip": "^1.1.5"
  }
}
```

__server.js__

This is the actuall expresss application used in testing the setup.

```
'use strict';

const express = require('express');
const ip = require('ip');

// Constants
const PORT = 8080;
// const HOST = '0.0.0.0';
const HOST = ip.address();
console.log(HOST)

// App
const app = express();
app.get('/', (req, res) => {
  res.send('Hello world\n');
});

app.listen(PORT, HOST);
console.log(`Running on http://${HOST}:${PORT}`);
```

__Dockerfile__, __Dockerfile.dev__, __Dockerfile.prod__

```
FROM node:boron
WORKDIR /usr/src/app
COPY package.json .
RUN npm install
COPY . .
EXPOSE 8000
CMD [ "npm", "start" ]
```

__docker-compose.dev.yml__

```
version: '3'
services:
  dev_web:
    # DEFINE BUILD CONTEXT.
    build:
        #context: .
        context: ../../
        # DOCKERFILE PATH RELATIVE TO BUILD CONTEXT.
        #dockerfile: ./conf/docker/Dockerfile.dev
        dockerfile: ./conf/docker/Dockerfile
    ports:
       #MAPPING -> "HOST:CONTAINER"
     - "50000:8080"
    container_name: ttg_webapp_dev
    hostname: ttg_webapp_dev
```

__docker-compose.prod.yml__

```
version: '3'
services:
  prod_web:
    # DEFINE BUILD CONTEXT.
    build:
        #context: .
        context: ../../
        # DOCKERFILE PATH RELATIVE TO BUILD CONTEXT.
        # dockerfile: ./conf/docker/Dockerfile.prod
        dockerfile: ./conf/docker/Dockerfile
    ports:
       #MAPPING -> "HOST:CONTAINER"
     - "52000:8080"
    container_name: ttg_webapp_prod
    hostname: ttg_webapp_prod
```

__docker-compose.network.yml__

```
version: '3'
services:
  net_web:
    # DEFINE BUILD CONTEXT.
    build:
        #context: .
        context: ../../
        # DOCKERFILE PATH RELATIVE TO BUILD CONTEXT.
        dockerfile: ./conf/docker/Dockerfile
    networks:
        net_net:
            ipv4_address: 10.0.0.9
    ports:
       #MAPPING -> "HOST:CONTAINER"
     - "54000:8080"
    container_name: ttg_webapp_net
    hostname: ttg_webapp_net
networks:
    net_net:
        ipam:
            driver: default
            config:
                - subnet: 10.0.0.0/8
```
