
Docker
│
├── Images            (Blueprints)
├── Containers        (Running Applications)
├── Volumes           (Persistent Storage)
├── Networks          (Communication)
├── Docker Compose    (Multiple Containers)
├── Build Cache       (Build Optimization)
└── System            (Cleanup & Information)


Docker
│
├──────────────────────────────────────────────
├── Images
│
│   List Images
│   ├── docker images
│   ├── docker image ls                                   same outputs both
│
│   Build Image
│   ├── docker build -t ts-server:latest .                built images with tag,reposirtoy_name ;version current_folder 
│   ├── docker build -t ts-server:v1 .
│   ├── docker build --no-cache -t ts-server .             it does not uses any cache build freshly 
│   ├── docker build -f Dockerfile.dev -t ts-server .       
     |---docker build -f Dockerfile.prod -t backend-prod .
       cretes multimer docker files for production or etc required separet dockr filees like this
         filename 
                       FROM node:22-alpine
 
                       WORKDIR /development

                      COPY package*.json ./

                      RUN npm install

                     COPY . .

                    EXPOSE 3000

                    CMD ["npm","run","dev"]



 
     docker build -q -t ts-server .                         only print image ids rather then all logs during building
│
│   Download Image
│   ├── docker pull nginx
│   ├── docker pull node:22-alpine
│
│   Tag Image
│   ├── docker tag ts-server:latest ts-server:v1
│   ├── docker tag ts-server:latest backend:production
│
│   Upload Image
│   ├── docker push username/ts-server:v1
│
│   Inspect Image
│   ├── docker image inspect ts-server
│
│   Image History
│   ├── docker history ts-server
│
│   Save Image
│   ├── docker save -o ts-server.tar ts-server
│
│   Load Image
│   ├── docker load -i ts-server.tar
│
│   Remove Image
│   ├── docker rmi ts-server
│   ├── docker image rm ts-server
│
│   Remove Dangling Images
│   ├── docker image prune
│
│
├──────────────────────────────────────────────
├── Containers
│
│   Create & Start
│   ├── docker run ts-server                                       run the container and attach to terminal to show logs                        
│   ├── docker run -d ts-server                                    run the container and only show container id in terminal
│   ├── docker run -it ubuntu bash
│   ├── docker run --name backend ts-server
│   ├── docker run -p 3000:3000 ts-server
│   ├── docker run -d --name backend -p 3000:3000 ts-server
│   ├── docker run -e PORT=5000 ts-server
│   ├── docker run -v mydata:/app/data ts-server
│
│   List Containers
│   ├── docker ps
│   ├── docker ps -a
│
│   Start Container
│   ├── docker start backend            starts an existing stoped continer
│   ├── docker start <container-id>     
│
│   Stop Container 
│   ├── docker stop backend               Please finish your work and shut down
│   ├── docker stop <container-id> 
│
│   Restart Container
│   ├── docker restart backend            stops and start an existing container
│
│   Kill Container
│   ├── docker kill backend                Forcefully stop a running container.
│
│   Logs
│   ├── docker logs backend
│   ├── docker logs -f backend
│   ├── docker logs --tail 50 backend
│
│   Execute Command
│   ├── docker exec -it backend sh
│   ├── docker exec -it backend bash
│   ├── docker exec backend ls
│
│   Inspect Container
│   ├── docker inspect backend
│
│   Rename Container
│   ├── docker rename backend api-server
│
│   Container Processes
│   ├── docker top backend
│
│   Resource Usage
│   ├── docker stats
│   ├── docker stats backend
│
│   Attach Terminal
│   ├── docker attach backend
│
│   Copy Files
│   ├── docker cp app.js backend:/app
│   ├── docker cp backend:/app/log.txt .
│
│   Remove Container
│   ├── docker rm backend
│   ├── docker rm -f backend
│
│   Remove Stopped Containers
│   ├── docker container prune
│
│
├──────────────────────────────────────────────
├── Volumes
│
│   Create Volume
│   ├── docker volume create mydata
│
│   List Volumes
│   ├── docker volume ls
│
│   Inspect Volume
│   ├── docker volume inspect mydata
│
│   Remove Volume
│   ├── docker volume rm mydata
│
│   Remove Unused Volumes
│   ├── docker volume prune
│
│
├──────────────────────────────────────────────
├── Networks
│
│   Create Network
│   ├── docker network create mynetwork
│
│   List Networks
│   ├── docker network ls
│
│   Inspect Network
│   ├── docker network inspect mynetwork
│
│   Connect Container
│   ├── docker network connect mynetwork backend
│
│   Disconnect Container
│   ├── docker network disconnect mynetwork backend
│
│   Remove Network
│   ├── docker network rm mynetwork
│
│   Remove Unused Networks
│   ├── docker network prune
│
│
├──────────────────────────────────────────────
├── Docker Compose
│
│   Start Services
│   ├── docker compose up
│   ├── docker compose up -d
│
│   Stop Services
│   ├── docker compose down
│
│   Restart
│   ├── docker compose restart
│
│   Logs
│   ├── docker compose logs
│   ├── docker compose logs -f
│
│   Execute Command
│   ├── docker compose exec web sh
│
│   Build
│   ├── docker compose build
│
│
├──────────────────────────────────────────────
├── Build Cache
│
│   View Cache
│   ├── docker builder ls
│
│   Remove Cache
│   ├── docker builder prune
│
│
├──────────────────────────────────────────────
└── System
    │
    Information
    ├── docker version
    ├── docker info
    ├── docker system df

    Cleanup
    ├── docker system prune
    ├── docker system prune -a

    Context
    ├── docker context ls
    ├── docker context use default







used commands 
docker run --rm -it docker_practice sh
