# Part 1 :: Exercises

## 1.1:
```
# docker ps -a

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                     PORTS               NAMES
d9215fbe691e        nginx               "/docker-entrypoin..."   2 minutes ago       Up 2 minutes               80/tcp              pensive_nightingale
bacbb1431ba5        nginx               "/docker-entrypoin..."   2 minutes ago       Exited (0) 2 seconds ago                       ecstatic_pare
05c799d1ffc7        nginx               "/docker-entrypoin..."   4 minutes ago       Exited (0) 9 seconds ago                       elegant_pike
```

## 1.2:
```
# docker ps -a

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES

# docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
```

## 1.3:  
```
Secret message is: 'You can find the source code here: https://github.com/docker-hy'
```

## 1.4:
```
# docker run -d -it --name curler ubuntu sh -c 'echo "Input website:"; read website; echo "Searching.."; sleep 1; curl http://$website;'
# docker exec -it curler bash
# apt-get -y update; apt-get -y install curl
# exit
# docker attach curler
# helsinki.fi
```

## 1.5:
```
# docker run -d -it --name alpine_test devopsdockeruh/simple-web-service:alpine
# docker images

devopsdockeruh/simple-web-service   ubuntu              4e3362e907d5        2 months ago        83MB
devopsdockeruh/simple-web-service   alpine              fd312adc88e0        2 months ago        15.7MB

# tail -f text.log
# docker exec -it alpine_test sh

Secret message is: 'You can find the source code here: https://github.com/docker-hy'
```

## 1.6:

1. Go with browser to docker hub and search devopsdockeruh/pull_exercise
2. The solution already reads under overview tab. On the right side under Source Repository there is a link to github.com to the source code and Dockerfile
In the terminal:
```
# docker run -it devopsdockeruh/pull_exercise
# basics
You found the correct password. Secret message is:
"This is the secret message"
```

## 1.7:

**Dockerfile**:
```
FROM devopsdockeruh/simple-web-service:alpine
CMD server
```
**Commands**:
```
# docker build . -t simple-web-service:web-server
# docker run simple-web-service:web-server
```

## 1.8:
**Dockerfile**:
```
FROM ubuntu:18.04
RUN apt-get -y update; apt-get -y install curl
CMD echo "Input website:"; read website; echo "Searching.."; sleep 1; curl http://$website;
```
**Commands**:
```
# sudo docker build . -t ubuntu:curler
# sudo docker run -it ubuntu:curler
# helsinki.fi
```
## 1.9:
```
# touch text.log && docker run -v "$(pwd)/text.log:/usr/src/app/text.log" devopsdockeruh/simple-web-service
```

## 1.10:
```
# docker run -p 1111:8080 devopsdockeruh/simple-web-service server
```
- Then point your browser to localhost:1111

## 1.11:

1. clone project: https://github.com/docker-hy/material-applications/tree/main/spring-example-project
2. cd into folder
3. create file Dockerfile:
```
FROM openjdk:8
COPY . /usr/src/myapp
WORKDIR /usr/src/myapp
RUN ./mvnw package
CMD ["java", "-jar","/usr/src/myapp/target/docker-example-1.1.3.jar"]
```
-- Then run following command
```
# docker build . -t spring-project && docker run -p 8080:8080 spring-project
```

## 1.12:

Clone the project: https://github.com/docker-hy/material-applications/tree/main/example-frontend     
and place Dockerfile in the project folder, Dockerfile:
```
FROM ubuntu:18.04
EXPOSE 5000
COPY . .
RUN apt-get update &&  apt-get -y install curl
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash
RUN apt install -y nodejs
RUN npm install
RUN npm run build
RUN npm install -g serve
CMD ["serve","-s","-l","5000","build"]
```
-- Run following command
```
# docker build . -t frontend-project && docker run -p 5000:5000 frontend-project
```
## 1.13:
**Dockerfile**:
```
FROM ubuntu:18.04
EXPOSE 8080
COPY . .
ENV PORT=8080
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/go/bin
ENV REQUEST_ORIGIN=http://localhost:5000
RUN apt-get update &&  apt-get -y install curl
RUN curl https://dl.google.com/go/go1.16.4.linux-amd64.tar.gz -o go1.16.4.linux-amd64.tar.gz
RUN rm -rf /usr/local/go && tar -C /usr/local -xzf go1.16.4.linux-amd64.tar.gz
RUN go build
CMD ["./server"]
```
-- Run following command
```
# docker build . -t backend-project && docker run -p 8080:8080 backend-project
```

## 1.14:
Dockerfile for frontend:
```
FROM ubuntu:18.04
EXPOSE 5000
ENV REACT_APP_BACKEND_URL=http://localhost:8080
COPY . .
RUN apt-get update &&  apt-get -y install curl
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash
RUN apt install -y nodejs
RUN npm install
RUN npm run build
RUN npm install -g serve
CMD ["serve","-s","-l","5000","build"]
```
Dockerfile for backend:
```
FROM ubuntu:18.04
EXPOSE 8080
COPY . .
ENV PORT=8080
ENV PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/go/bin
ENV REQUEST_ORIGIN=http://localhost:5000
RUN apt-get update &&  apt-get -y install curl
RUN curl https://dl.google.com/go/go1.16.4.linux-amd64.tar.gz -o go1.16.4.linux-amd64.tar.gz
RUN rm -rf /usr/local/go && tar -C /usr/local -xzf go1.16.4.linux-amd64.tar.gz
RUN go build
CMD ["./server"]
```
-- Run following commands each in their own terminal windows
```
# docker run -p 5000:5000 -it frontend-project
# docker run -p 8080:8080 -it backend-project
```

## 1.15:
https://hub.docker.com/r/mcprn/whatip


Done exercises 15/16. Skipped exercise is **1.16: Heroku**, which was **NOT** mandatory exercise.
