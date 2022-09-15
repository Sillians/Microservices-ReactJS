## Docker - ReactJS (Development workflow for a Simple React-App using Docker containers)

Docker is an open source software containerization platform that allows you to package applications into standardized, isolated units called containers. These containers combine the applications source code with the operating system libraries and dependencies required to run that code in any environment. Docker allows you to build, test, deploy, and scale applications quickly. Its development speed, scalability, and ecosystem richness are some of the many reasons why we will use it to containerize a React application. 


### Install Node
- `sudo apt update`
- `sudo apt install nodejs npm`
    - `nodejs --version`
- `sudo apt-get install -y nodejs`
    - `node --version`
    - `npm --version`
- `sudo apt-get install gcc g++ make`
- `sudo apt install build-essential`


### Initialize a Dummy React App
- `npx create-react-app .`

### Start the development server by
- Run `npm start`

### Build an image with Dockerfile
- Use a `Node` image to build a Docker image
- build image `docker build -t react-image .`
- list images `docker image ls`
- Run the container
    - `docker run -it -d --name react-app react-image`
    - check running containers `docker ps`
    - kill the container `docker rm react-app -f` since the container isn't running on the supposed port.
    - Run `docker run -it -d -p 3000:3000 --name react-app react-image`   to specify the port you are sending traffic to in the container, 3000:3000 local machine port traffic and container port traffic
    - Forwarding traffic from port `3000` in the localhost to port `3000` in the docker container.
- Connect into the file system of the Docker container
    - Run `docker exec -it react-app bash` 

### .dockerignore
- We don't need the `Dockerfile` and `node_modules` in the container, and some certain files we don't want to copy into our container.
- create the `.dockerignore` file and specify the files we don't want to be copied into the container.
- kill the current running container `docker rm react-app -f`
- Rebuild the image
    - `docker build -t react-image .`
- Run the container
    - `docker run -it -d -p 3000:3000 --name react-app react-image`
- bash into the container
    - `docker exec -it react-app bash`


### Making Changes that gets propagated/Updated into the Docker Container

- Making changes to your source file and having to rebuild and run your container each time, thats not a scalable and maintable solution, which slows development time.
- To optimize this, we make use of volumes, which allows us to have persistent data
    - Using the `bind mount` volume
    - Allows you to take a file / folder in your host machine and mount/sync it in your docker container
    - using the `-v` which stands for volume
    - `-v dirlocaldirectory:containerdirectory`
    - `docker run -it -v ${pwd}/src:/app/src -d -p 3000:3000 --name react-app react-image`
    - To make the bind mount `read-only` you add the `:ro` to the volume flag, our container could read-only and not make changes to our host files. `docker run -it -v ${pwd}/src:/app/src:ro -d -p 3000:3000 --name react-app react-image`
    - The `-v $(pwd):/src` option tells Docker Daemon to start up a volume in our container. Volumes are the best way to persist data in Docker. Here we are telling that we want the current directory - retrieved from `$(pwd)` - to be added to our container in the folder `/src`.

### Working with Environment Variables
- Can identify your env. variables in the Dockerfile or while running your container using the `-e` flag
- If you have tons of env. variables, you can create a `.env` file, and have all your env. variables in there.
- To pass the env variable files whilst running your container, 
    - `docker run -it --env-file ./.env -v ${pwd}/src:/app/src -d -p 3000:3000 --name react-app react-image`


### Docker Compose
- Docker compose is used to run multiple containers as a single service.
- It provides a way to orchestrate multiple containers that work together.
- create a new file, `docker-compose.yml`
- `services` represent docker container, so each container that you have is going to represent a different service.
- Run `docker-compose up -d` to create and start the container(s) (`-d` flag for `detached` mode)
- Run `docker-compose down` to stop and remove containers, networks.
- The `-t` (or `--tty`) flag tells docker to allocate a virtual terminal session within the container. This is commonly used with the `-i` (or --interactive) option, which keeps STDIN open even if running in detached mode.
- To trigger a build again, attach the `--build` flag
    - `docker-compose up -d --build`


### Docker Compose for Dev, Stag and Prod Environment
#### Stages (Out of Context)
- development is what you have on your local machine
- stage: not stable, built with the last contributions from git
- QA: same data as production; QA is really important, as we usually clone the filesystem and database of production and use this environment to test what will happen when we build the backend with a Git tag. We can update QA with the tag, run tests, have some people log in to check if everything is ok then we can rebuild the container for production.
- production: roll-outs to the production environment

`TODO` : Attach the dev, stag, prod image


- `npm run build` for prod
- Set up a web server that would help to serve the static files
- Using Multi-Stage Docker builds for Production with NGINX
- We would build the image, copy the result of the build(`npm run build`) into the `NGINX` container.
- Create `Dockerfile.dev` and `Dockerfile.prod`
- To build any of the image(`.dev or .prod`), use the `-f` flag and specify the particular dockerfile to build.
    - `docker build -f Dockerfile.prod -t docker-image-prod .`
- nginx is opened up on port `:80` , we use port `8080` for our host machine
    - `docker run --env-file ./.env -d -p 8080:80 --name react-app-prod docker-image-prod`

- Create `docker-compose` files for `prod` and `dev`.
- Run `docker-compose -f docker-compose.yml -f docker-compose-dev.yml up -d --build` to start the development stage build.
- Run `docker-compose -f docker-compose.yml -f docker-compose-prod.yml up -d --build` and test with port `8080`
- `ARGUMENTS` allows us to pass data from a Docker Compose file into a Docker Build File
- Tear down and rebuild after passing the arguments.




