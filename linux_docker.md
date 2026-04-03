# for ec2: System Requirements:

Ubuntu
t2 Medium/large
Active ssh, http and HTTPS
storage: 30gb min

# then in ec2:

    sudo apt-get update

    then
    sudo apt-get install docker.io

    to check the installation status:
        sudo systemctl status docker

        but right now, you will not have permission to see docker files and images... as ubuntu user does not have these permissions..

        so
        sudo usermod -aG docker $USER

        then

        newgrp docker


        now do docker login

        with PAT (persona access token) or password
                thiS is done so that you can pull or push the docker images and containers to dockerhub

        DOCKERFILE:

            the format of this file will depend on the program that you are trying to build and the languarge that you have used to write your code...

            the file name is always mentioned in starting capital letter and no extensions.. (Dockerfile)

            despite having no fixed format, there are certain convensions of writting a dockerfile that always remains the same...


                    FROM source_name (pull a base image which gives all required libraries)

                    WORKDIR the_name_of_dir_you_want_inside_the_container (eg.. /app)

                    COPY what_files_you_need_to_copy_from_your_code (eg.. COPY . .)
                            (generally we remove the dependency files like npm etc before creating this file as they are heavy files and no need to copy)

                    RUN the_packages_you_need_for_running_that_program (eg.. RUN npm install)

                    EXPOSE only_if_there_is_any_port_to_expose (eg.. EXPOSE 3000)

                    CMD [command_seperated_by_comma_to_start_the_app] (eg.. CMD ["npm", "start"])

            in the above dockerfile, starting from FROM to RUN, these all commands form layers in the container... only the CMD command is used to run the app inside the container..

            So, CMD can be overwritten when we run the command Docker run in the terminal.
            There is another such command called ENTRYPOINT which is same as CMD, except that it cannot be overwritten

        now the dockerfile is ready... now you have to build an image of this file... so you need to run the command in the terminal:

            make sure you are in the same directory where docker file exists..

                docker build -t image_name .

            now the image is created... you can check with
                docker images

        now the image has been built, now to run the image we simply type:
                docker run -d image_name
                    (-d) is the detached mode

            remember: everytime you change the source code in your system, you will need to rebuild the image and run docker again..

        however, the earlier works only when there is no port requiremnt. If in case the app runs on a port, then the port needs to bind with docker port.
                docker run -p host_port:container_port image_name
                    eg... docker run -d -p 3000:3000 node_backend

            now check if the app is running in the background...
                docker ps

            now you can go to aws instance copy the public ip and paste it in browser (https://ip) and check if the app is running...
            TROUBLESHOOT STEPS:
            however, if in case there are issues you can check the logs..
                docker logs container_id

            there could be one problem while using aws, in port binding... as port 3000 may not be available for everyone at a single time...
                so in your aws instance..
                    -> go to security -> go to security groups -> edit inbound rules -> add rule -> custom tcp, info 80, source- anywhere  -> save rule

            remember: at any point to stop a container:
                            docker stop container_id
                      to start a stopped container:
                            docker start container_id

# DOCKER NETWORK:

    this assists in communication of one container to another...

    There are mainly 7 types of docker network:
        1. Host
        2. Bridge (default)
        3. User Defined Bridge (Custom Bridge)
        4. None
        5. MCVLAN
        6. IPVLAN
        7. Overlay

        the last three types are generally used for docker swarm  - means docker in cluster eg.. musliple containers accross multiple machines..
        this is not so used now a days.. as it is generally achived through kubernates..


        now to check types of network in your linux system...
            docker network ls

        to create a nework:
            docket network create -d bridge mynetwork

        the main use of docker networks are in two-tier and three tier apps..

        remember: at any point to to pass env variables:
            docker run -e

        example:
        suppose you have a node js two tier backend application with ejs that calls database mongodb to store tasks and dates...

        so after cloning the repo from github...

        pull mongodb from dockerhub...
            docker pull mongo:7

        now

        step 1: build the two tier app image with dockerfile and then
            docker build -t two-tier-app .

        step 2: create a custom bridge network
            docker network create -d bridge two-tier

        step 3: run docker mongodb with two tier
            docker run -d --name mongodb --network two-tier -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=admin123 mongo:7

        step 4: check MongoDB is running
            docker logs mondodb

        step 5: run your app with correct en variables
            docker run -d --name two-tier-node-app -p 80:3000 -e MONGO_URI="mongodb://admin:admin123@mongodb:27017" two-tier-app

# DOCKER COMPOSE:

        the purpose of this file is to run multiple containers with a single file... this type of file is called yml or yaml type of file (Yet Another Mark-up Language)

            eg..

            step 0: install docker compose on ec2:
                sudo apt-get install docker-compose-v2

            Step 1: choose version:
                version: "3.8"

            step 2: services: this defines the no. of containers..
                services:
                    frontend:
                        container_name: frontend
                        build:
                            context: .
                        ### [need to check with chatgpt on other key values as the frontend will require npm run build and nginx....]
                        image: three-tier-frontend
                        volumes:
                            - "...."
                        environment:
                            api: "...."
                        ports:
                            - "80:80"
                        networks:
                            three-tier:
                        depends_on:
                            - backend
                                ### this is to specify that the container will depend on backend to run first and then it will start... otherwise there will be errors..
                        healthcheck:
                            #### [test with the cmd command to ping.. no idea of the command now, will check with chatgpt...]
                                ### this is to work in tandem with "depends on" key of frontend compose code, so that the frontend only starts running after the backend has started error free....
                        restart: always
                            ### if health check does not pass then the container will attempt to restart


                    backend:
                        container_name: backend
                        image: three-tier-backend
                        env_file: ./backend/.env
                        network:
                            three-tier:
                        healthcheck:
                            test: [ #### test with the cmd command to ping.. no idea of the command now, will check with chatgpt...]
                                ### this is to work in tandem with "depends on" key of frontend compose code, so that the frontend only starts running after the backend has started error free....
                        restart: always
                            ### if health check does not pass then the container will attempt to restart



            step 3: name all the networks and volumes that you have created for the containers:
                volumes:
                    mongodb:

                networks:
                    three-tier

            step 4: simply save and exit the yml file

            step 5: no need to build and run anything... you can simply command:
                docker compose up -d

    CHATGPT EXPALANTION ON COMPOSE:

        DOCKER COMPOSE

            Purpose:
            Docker Compose is used to define and run multiple containers as one application using a single YAML file.

            Important:
            Compose does not replace Dockerfiles.
            Compose uses Dockerfiles to build images, then runs containers together.

            YAML:
            YAML means "Yet Another Markup Language".
            It is a human-readable configuration format.
            It uses indentation, not curly brackets.

            Main idea:
            Instead of running many docker build and docker run commands manually,
            we define everything in docker-compose.yml and then run one command.

            --------------------------------------------------
            Step 0: Install Docker Compose on EC2
            --------------------------------------------------
            sudo apt update
            sudo apt install docker-compose-plugin -y

            Use command:
            docker compose
            (not docker-compose)

            --------------------------------------------------
            Step 1: Create backend .env after cloning
            --------------------------------------------------
            backend/.env

            Example:
            PORT=5000
            NODE_ENV=production
            MONGO_URI=your_mongodb_atlas_connection_string
            CORS_ORIGIN=http://YOUR_EC2_PUBLIC_IP

            --------------------------------------------------
            Step 2: Compose file structure
            --------------------------------------------------
            services:
            frontend:
            backend:

            networks:
            three-tier:

            --------------------------------------------------
            Step 3: services
            --------------------------------------------------
            A service defines one container.

            Example:
            services:
            backend:
            frontend:

            --------------------------------------------------
            Step 4: build
            --------------------------------------------------
            build:
            context: ./backend

            This tells Compose to build the image using the Dockerfile inside ./backend.

            --------------------------------------------------
            Step 5: container_name
            --------------------------------------------------
            container_name: backend

            This sets a fixed name for the container.

            --------------------------------------------------
            Step 6: env_file
            --------------------------------------------------
            env_file:
            - ./backend/.env

            This loads environment variables into the container.

            --------------------------------------------------
            Step 7: ports
            --------------------------------------------------
            ports:
            - "80:80"

            This maps host port 80 to container port 80.

            frontend needs this because browser must access it.
            backend does not need ports in this architecture because it stays internal.

            --------------------------------------------------
            Step 8: networks
            --------------------------------------------------
            networks:
            - three-tier

            This connects the service to a Docker network.
            Containers on the same network can communicate with each other.

            --------------------------------------------------
            Step 9: depends_on
            --------------------------------------------------
            depends_on:
            backend:
                condition: service_healthy

            This means frontend waits until backend becomes healthy.

            --------------------------------------------------
            Step 10: healthcheck
            --------------------------------------------------
            healthcheck:
            test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:5000/health"]
            interval: 10s
            timeout: 5s
            retries: 5
            start_period: 10s

            Meaning:
            - test = command to check health
            - interval = run every 10 seconds
            - timeout = fail if test takes more than 5 seconds
            - retries = mark unhealthy after 5 failures
            - start_period = wait 10 seconds before counting failures

            For frontend:
            healthcheck:
            test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost"]

            --------------------------------------------------
            Step 11: restart
            --------------------------------------------------
            restart: always

            This restarts the container automatically if it stops.

            --------------------------------------------------
            Step 12: define networks at bottom
            --------------------------------------------------
            networks:
            three-tier:
                driver: bridge

            --------------------------------------------------
            Step 13: start everything
            --------------------------------------------------
            docker compose up -d --build

            This:
            - builds images
            - creates network
            - starts backend
            - starts frontend

            --------------------------------------------------
            Step 14: check status
            --------------------------------------------------
            docker compose ps

            --------------------------------------------------
            Step 15: check logs
            --------------------------------------------------
            docker compose logs -f
            docker compose logs -f backend
            docker compose logs -f frontend

            --------------------------------------------------
            Step 16: stop and remove containers
            --------------------------------------------------
            docker compose down


# DOCKER LOGS:

        to check log files of the conainer...
            docker logs container_name/id

        TO CONSTANTLY SEND LOGS TO A FILE IN BACKGROUND YOU CAN USE:
            nohup docker attach container_id &
                This will create a file called nohup.out and keep storing the logs in that file...


















