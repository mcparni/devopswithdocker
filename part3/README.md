# Part 3 :: Exercises
## 3.1:
Repository with the deployment pipeline config:  
https://github.com/mcparni/whatip

## 3.3:
**frontend**:  
**!Note** https://github.com/docker-hy/material-applications/tree/main/example-frontend needs to be in same folder as this **Dockerfile**:
```
FROM ubuntu:18.04
WORKDIR /usr/app
EXPOSE 5000
COPY . .
RUN apt-get update &&  apt-get -y install curl
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash
RUN apt install -y nodejs
RUN useradd -m appuser
RUN chown -R appuser:appuser /usr/app
RUN chown -R appuser:appuser /usr/lib/node_modules
USER appuser
RUN cd /usr/app
RUN npm install
RUN npm run build
USER root
RUN npm install -g serve
USER appuser
RUN cd /usr/app
CMD ["serve","-s","-l","5000","build"]
```
**backend**:
**!Note** https://github.com/docker-hy/material-applications/tree/main/example-backend needs to be in same folde as this **Dockerfile**:
```
FROM ubuntu:18.04
WORKDIR /usr/app
EXPOSE 8080
COPY . .
ENV PORT=8080
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/go/bin
ENV REQUEST_ORIGIN=http://localhost:5000
RUN apt-get update &&  apt-get -y install curl
RUN curl https://dl.google.com/go/go1.16.4.linux-amd64.tar.gz -o go1.16.4.linux-amd64.tar.gz
RUN rm -rf /usr/local/go && tar -C /usr/local -xzf go1.16.4.linux-amd64.tar.gz
RUN useradd -m appuser
RUN chown -R appuser:appuser /usr/app
USER appuser
RUN cd /usr/app
RUN go build
CMD ["./server"]
```
## 3.4:
Original image sizes:  
Frontend 	517MB  
Backend		906MB  

New Dockerfiles:  
**Frontend Dockerfile**, (image size 456MB):  
```
FROM ubuntu:18.04  
WORKDIR /usr/app
EXPOSE 5000
COPY . .
RUN apt-get update && apt-get -y install curl && curl -sL https://deb.nodesource.com/setup_14.x | bash && \
        apt install -y nodejs && useradd -m appuser && npm install -g serve && \
        chown -R appuser:appuser /usr/app && chown -R appuser:appuser /usr/lib/node_modules && \
        apt-get purge -y --auto-remove curl && \ 
        rm -rf /var/lib/apt/lists/*
USER appuser
RUN cd /usr/app && npm install &&  npm run build
CMD ["serve","-s","-l","5000","build"]
```

**Backend Dockerfile**, (image size 602MB):
```
FROM ubuntu:18.04
WORKDIR /usr/app
EXPOSE 8080
COPY . .
ENV PORT=8080
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/go/bin
ENV REQUEST_ORIGIN=http://localhost:5000
RUN apt-get update &&  apt-get -y install curl && curl https://dl.google.com/go/go1.16.4.linux-amd64.tar.gz -o go1.16.4.linux-amd64.tar.gz && \
        rm -rf /usr/local/go && tar -C /usr/local -xzf go1.16.4.linux-amd64.tar.gz && rm -rf /usr/app/go1.16.4.linux-amd64.tar.gz && \
        useradd -m appuser && \
        apt-get purge -y --auto-remove curl && rm -rf /var/lib/apt/lists/* && \
        cd /usr/app && go build && chown -R appuser:appuser /usr/app
USER appuser
CMD ["./server"]
```
## 3.5:

Original image sizes:  
Frontend 	456MB  
Backend		602MB  

**frontend Dockerfile**:
```
FROM node:14-alpine3.12
WORKDIR /usr/app
EXPOSE 5000
COPY . .
RUN cd /usr/app && npm install &&  npm run build && npm install -g serve && \
adduser -D appuser && chown -R appuser:appuser /usr/app
USER appuser
CMD ["serve","-s","-l","5000","build"]
```
New frontent image size: 345MB  

**backend Dockerfile**:
```
FROM golang:1.17.3-alpine3.13
WORKDIR /usr/app
EXPOSE 8080
COPY . .
ENV PORT=8080
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/go/bin
ENV REQUEST_ORIGIN=http://localhost:5000
RUN cd /usr/app && go build && adduser -D appuser && chown -R appuser:appuser /usr/app
USER appuser
CMD ["./server"]
```
New backend image size: 467MB

## 3.6:
frontend-project image size before multistage build: 345MB  
frontend-project image size after multistage build: 128MB 
**frontend-project dockerfile**:
```
FROM node:14-alpine3.12 as frontend-build-stage
WORKDIR /usr/app
COPY . .
RUN cd /usr/app && npm install &&  npm run build

FROM node:14-alpine3.12
WORKDIR /usr/app
EXPOSE 5000
COPY --from=frontend-build-stage /usr/app/build/ /usr/app/build
RUN npm install -g serve && \
adduser -D appuser && chown -R appuser:appuser /usr/app
USER appuser
CMD ["serve","-s","-l","5000","build"]
```
backend-project image size before multistage build: 467MB   
backend-project image size after multistage build: 23.3MB   
**backend-project dockerfile**:
```
FROM golang:1.17.3-alpine3.13 as backend-build-stage
WORKDIR /usr/app
COPY . .
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/go/bin
RUN cd /usr/app && go build

FROM alpine:latest
WORKDIR /usr/app
EXPOSE 8080
COPY --from=backend-build-stage /usr/app/server /usr/app/server
ENV PORT=8080 \
  REQUEST_ORIGIN=http://localhost:5000
CMD ["./server"]
```
## 3.7:

Original **WhatIp Dockerfile** (image size: 277MB):
```
FROM ubuntu:18.04
EXPOSE 3000
COPY . .
ENV PORT=3000
RUN apt-get update &&  apt-get -y install curl
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash
RUN apt install -y nodejs
RUN npm install
CMD ["node","index.js"]
```
**WhatIp Dockerfile** after multistage build and optimizations (image size: 126MB):
```
FROM node:14-alpine3.12 as whatip-build-stage
WORKDIR /usr/app
COPY . .
RUN npm install

FROM node:14-alpine3.12
WORKDIR /usr/app
EXPOSE 3000
ENV PORT=3000
COPY --from=whatip-build-stage /usr/app/ /usr/app
RUN adduser -D appuser && chown -R appuser:appuser /usr/app
USER appuser
CMD ["node","index.js"]
```
## 3.8
Diagram shows a cluster which has a load balancer. Enduser can send requests only to load balancer. Load balancer serves front end according to load. Front end has volume mounted for files. front ends can talk with backend. Backends talk with front end and can also access to database. Backends have mounted the volume too for persistent database and other files.
![Kubernetes cluster](https://raw.githubusercontent.com/mcparni/devopswithdocker/main/part3/kubernetes.png)



Done exercises 7/8. Skipped exercise is **3.2**, which was **NOT** mandatory exercise.



