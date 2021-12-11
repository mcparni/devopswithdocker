2.1:

docker-compose.yml:

version: '3.5' 

services: 

    simple-web-service: 
      image: devopsdockeruh/simple-web-service
      build: . 
      volumes: 
        - ./text.log:/usr/src/app/text.log
      container_name: the-sws

2.2:

docker-compose.yml:

version: '3.5' 

services: 

    simple-web-service: 
      image: devopsdockeruh/simple-web-service
      build: . 
      command: server
      ports: 
        - 8080:8080 
      container_name: the-sws-2

2.3:

# I already have the images done in previous exercise set:
# frontend-project and backend-project. Following solution
# assumes they are present.

docker-compose.yml:

version: '3.5' 

services: 
    frontend-service: 
      image: frontend-project:latest
      environment: 
        - REACT_APP_BACKEND_URL=http://localhost:8080
      command: serve -s -l 5000 build
      ports: 
        - 5000:5000 
      container_name: frontend
    backend-service: 
      image: backend-project:latest
      environment: 
        - REQUEST_ORIGIN=http://localhost:5000
        - PORT=8080
        - PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/go/bin
      command: ./server
      ports: 
        - 8080:8080 
      container_name: backend

2.4:

docker-compose.yml:

version: '3.5' 

services: 
    frontend-service: 
      image: frontend-project:latest
      environment: 
        - REACT_APP_BACKEND_URL=http://localhost:8080
      command: serve -s -l 5000 build
      ports: 
        - 5000:5000 
      container_name: frontend
    backend-service: 
      image: backend-project:latest
      environment: 
        - REDIS_HOST=redis
        - REQUEST_ORIGIN=http://localhost:5000
        - PORT=8080
        - PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/go/bin
      command: ./server
      ports: 
        - 8080:8080 
      container_name: backend
    redis-service:
      image: redis
      ports: 
          - 6379:6379
      container_name: redis


2.5:
I hope I got this right, but here goes:

I just ran this commmand:
docker-compose up --scale compute=3

I opened up three browser windows and clicked 'Press here to test your solution' on each of them. Then I got 'Congratulations' text next to button.



2.6:

docker-compose.yml:

version: '3.5' 

services: 
    frontend-service: 
      image: frontend-project:latest
      environment: 
        - REACT_APP_BACKEND_URL=http://localhost:8080
      command: serve -s -l 5000 build
      ports: 
        - 5000:5000 
      container_name: frontend
    backend-service: 
      image: backend-project:latest
      environment: 
        - POSTGRES_HOST=db-service
        - POSTGRES_USER=postgres
        - POSTGRES_PASSWORD=postgres
        - REDIS_HOST=redis
        - REQUEST_ORIGIN=http://localhost:5000
        - PORT=8080
        - PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/go/bin
      command: ./server
      ports: 
        - 8080:8080 
      container_name: backend
    redis-service:
      image: redis
      ports: 
          - 6379:6379
      container_name: redis
    db-service:
      image: postgres
      restart: always
      environment:
        - POSTGRES_USER=postgres
        - POSTGRES_PASSWORD=postgres

2.7:
docker-compose.yml:

version: '3.5'

services:
  training-service:
    build: ./ml-kurkkumopo-training
    volumes:
      - training-model:/src/model
      - ./imgs:/src/imgs
  backend-service:
    build: ./ml-kurkkumopo-backend
    volumes:
      - training-model:/src/model
    depends_on:
      - training-service
    ports:
      - 5000:5000
  frontend-service:
    build: ./ml-kurkkumopo-frontend
    ports:
      - 3000:3000
volumes:
  training-model:


2.8:
Following nginx.conf is required:

events { worker_connections 1024; }

  http {
    server {
      listen 80;

      location / {
        proxy_pass http://frontend-service:5000/;
      }

      location /api/ {
        proxy_set_header Host $host;
        proxy_pass http://backend-service:8080/;
      }
    }
  }
      
docker-compose.yaml:

version: '3.5' 

services: 
    frontend-service: 
      image: frontend-project:latest
      environment: 
        - REACT_APP_BACKEND_URL=http://localhost:8080
      command: serve -s -l 5000 build
      ports: 
        - 5000:5000 
      container_name: frontend-service
    backend-service: 
      image: backend-project:latest
      environment: 
        - POSTGRES_HOST=db-service
        - POSTGRES_USER=postgres
        - POSTGRES_PASSWORD=postgres
        - REDIS_HOST=redis
        - REQUEST_ORIGIN=http://localhost:5000
        - PORT=8080
        - PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/go/bin
      command: ./server
      ports: 
        - 8080:8080 
      container_name: backend-service
    redis-service:
      image: redis
      ports: 
          - 6379:6379
      container_name: redis
    db-service:
      image: postgres
      restart: always
      environment:
        - POSTGRES_USER=postgres
        - POSTGRES_PASSWORD=postgres
    web:
      image: nginx
      volumes:
        - ./nginx.conf:/etc/nginx/nginx.conf
      ports: 
          - 80:80
      depends_on:
        - frontend-service
        - backend-service


2.9:
docker-compose.yaml:

version: '3.5'

services:
    frontend-service:
      image: frontend-project:latest
      environment:
        - REACT_APP_BACKEND_URL=http://localhost:8080
      command: serve -s -l 5000 build
      ports:
        - 5000:5000
      container_name: frontend
    backend-service:
      image: backend-project:latest
      environment:
        - POSTGRES_HOST=db-service
        - POSTGRES_USER=postgres
        - POSTGRES_PASSWORD=postgres
        - REDIS_HOST=redis
        - REQUEST_ORIGIN=http://localhost:5000
        - PORT=8080
        - PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/go/bin
      command: ./server
      ports:
        - 8080:8080
      container_name: backend
    redis-service:
      image: redis
      ports:
          - 6379:6379
      container_name: redis
    db-service:
      image: postgres
      restart: always
      environment:
        - POSTGRES_USER=postgres
        - POSTGRES_PASSWORD=postgres
      volumes:
        - ./database:/var/lib/postgresql/data

2.10:
The main fix here was the cors issue. Before the localhost:5000 was allowed, now with nginx the port 5000 is omitted thus it is different origin. Changing request origin http://localhost:5000 -> http://localhost fixed the buttons. That is changing the REQUEST_ORIGIN environment variable in backend. So returning nginx.conf file and docker-compose.yml and it is working.

nginx.conf:
 
events { worker_connections 1024; }

  http {
    server {
      listen 80;

      location / {
        proxy_pass http://frontend-service:5000/;
      }

      location /api/ {
        proxy_set_header Host $host;
        proxy_pass http://backend-service:8080/;
      }
    }
  }

docker-compose.yaml:

version: '3.5' 

services: 
    frontend-service: 
      image: frontend-project:latest
      environment: 
        - REACT_APP_BACKEND_URL=http://localhost:8080
      command: serve -s -l 5000 build
      ports: 
        - 5000:5000 
      container_name: frontend-service
    backend-service: 
      image: backend-project:latest
      environment: 
        - POSTGRES_HOST=db-service
        - POSTGRES_USER=postgres
        - POSTGRES_PASSWORD=postgres
        - REDIS_HOST=redis
        - REQUEST_ORIGIN=http://localhost
        - PORT=8080
        - PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/usr/local/go/bin
      command: ./server
      ports: 
        - 8080:8080 
      container_name: backend-service
    redis-service:
      image: redis
      ports: 
          - 6379:6379
      container_name: redis
    db-service:
      image: postgres
      restart: always
      environment:
        - POSTGRES_USER=postgres
        - POSTGRES_PASSWORD=postgres
      volumes:
        - ./database:/var/lib/postgresql/data
    web:
      image: nginx
      volumes:
        - ./nginx.conf:/etc/nginx/nginx.conf
      ports: 
          - 80:80
      depends_on:
        - frontend-service
        - backend-service

2.11:
NOT DONE (not mandatory). This is the only assignment from this set that is not done. 




