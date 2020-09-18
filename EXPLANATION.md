# 1. Choice of the base image on which to build each container.

I have a total of 4 containers that successed the project
This solution is self sustaining.

1. backend: node:latest
2. client: slim build of react scripts and packaged into nginx:stable-alpine
3. mongodbc mongo:latest - to have an independed db container
4. mongoxpress mongo-express:latest - for database administration


_____________________________________
# 2. Dockerfile directives used in the creation and running of each container.

## 2.1. backend:
FROM node:latest
WORKDIR /usr/backend
COPY package.json ./
RUN npm install
COPY . .


## 2.2 client:
build environment
FROM node:13.12.0-alpine as build
WORKDIR /app
ENV PATH /app/node_modules/.bin:$PATH
COPY package.json ./
COPY package-lock.json ./
RUN npm ci --silent
RUN npm install react-scripts@3.4.1 -g --silent
COPY . ./
RUN npm run build

- production environment
FROM nginx:stable-alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]


_____________________________________
# 3. Docker-compose Networking (Application port allocation and a bridge network implementation) where necessary.
- Network selection was the default : yolo_default
- running on local and exposed to the host


## 3.1 backend
ports:
- 5000:5000


## 3.2 mongodb
ports:
- "27017:27017"


## 3.3 Client
ports:
- 3333:80


## 3.4 mongoxpress
ports:
- "8081:8081"


_____________________________________
# 4. Docker-compose volume definition and usage (where necessary).
volumes:
    mongodb_data_container:


## 4.1 mongodbc - for persistence
volumes:
        - ./.docker/mongodb/mongod.conf:/etc/mongod.conf
        - ./.docker/mongodb/initdb.d/:/docker-entrypoint-initdb.d/
        - ./.docker/mongodb/data/db/:/data/db/
        - ./.docker/mongodb/data/log/:/var/log/mongodb/


## 4.2 mongoxpress - for file sharing with mongodbc and configs
volumes:
        - ./.docker/mongodb/mongod.conf:/etc/mongod.conf


_____________________________________
# 5. Git workflow used to achieve the task.
- this was not achieved since the deployment approach i used is experimental


_____________________________________
# 6. Successful running of the applications and if not, debugging measures applied.
Debugging was mandated to enable the mongodb to run that included
 - creation of mongo.config
 - initdb.d script for initializing db
 - mongodb log file and making it read_write
 - .env variables file to protect the passwords
 - creation of eviromental variables to pass to the backend  server.js file

## 6.1 backend service environment:
         MONGODB_URI: mongodb://${MONGO_INITDB_ROOT_USERNAME}:${MONGO_INITDB_ROOT_PASSWORD}@mongodbc:27017?retryWrites=true&w=majority

## 6.2 mongodbc service environment:
           MONGO_INITDB_ROOT_USERNAME: ${MONGO_INITDB_ROOT_USERNAME}
           MONGO_INITDB_ROOT_PASSWORD: ${MONGO_INITDB_ROOT_PASSWORD}
           MONGO_INITDB_DATABASE: ${MONGO_INITDB_DATABASE}



_____________________________________
# 7. Good practices such as Docker image tag naming standards for ease of identification of images and containers.
- the model used for this deployment is not for availing a container to be pulled but rather scripts that successfully create a fully working and independed environment on a VM.

================================================
