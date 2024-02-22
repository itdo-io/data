---
title: 'Installing Docker and Basics to Get Started'
description: 'This guide will walk you through on how to installation of Docker, introduce you to basic concepts, and help you get started with how to run Docker containers and using Docker Compose.'
slug: 'installing-docker-basics-to-get-started'
# image: '/posts/installing-docker-basics-to-get-started/docker-c43rndw.jpg'
# image: 'https://res.cloudinary.com/dvpszpx3u/image/upload/c_crop,w_800,h_600/v1708456677/itdo.io/hbibdaosibyoyyxl1uzt.jpg'
category: 
    - docker
published: '2024-02-21'
---

## Table of Contents

Docker is a powerful platform for developing, shipping, and running applications. It allows you to separate your applications from your infrastructure so you can deliver software quickly. Docker packages software into standardized units called containers that have everything the software needs to run including libraries, system tools, code, and runtime. By doing so, it allows applications to run in various environments consistently.

## Installing Docker

The most accurate and secure way to install Docker is by following the official documentation, which is regularly updated. You can find the installation guide for your specific operating system at [Docker's official installation docs](https://docs.docker.com/engine/install/).

However, for most users, the quickest way to install Docker is by executing the following commands in your terminal:

```shell:terminal
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

Source: [GitHub - docker/docker-install](https://github.com/docker/docker-install)

It's important to note that the `get-docker.sh` script supports a limited number of distributions and might not include CentOS (though this could change in the future). If the script does not work for your distribution, refer back to the official documentation for alternative installation methods.

The above will also install docker compose. Docker Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services, networks, and volumes, and then create and start all the services from your configuration with a single command.

## Post-Installation Steps

To use Docker as a non-root user, you should follow the post-installation steps in the Docker documentation. For Linux users, the commands are as follows:

```shell:terminal
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

This allows you to run Docker commands without `sudo`.

## Testing Docker Installation

To verify that Docker has been installed correctly and is working, run the following command:

```shell:terminal
docker run hello-world
```

This command downloads a test image and runs it in a container. If the container executes correctly, you'll see a message indicating that your Docker installation is working.

## Docker Basics with Commands

Let's explore some basic Docker commands with an example. Consider you want to run a Node.js application in a Docker container:

```shell:terminal
docker run -d --name my-node-app \
  -p 8080:3000 \
  -v /home/user/app-data:/app/data \
  -e SECRET_KEY=some_secret_value \
  --network my-custom-network \
  node:16-alpine
```

**Explanation:**

- `docker run`: Starts a new container from an image.
- `-d`: Detached mode. Runs the container in the background.
- `--name my-node-app`: Names the container for easier reference.
- `-p 8080:3000`: Maps port 8080 on the host to port 3000 inside the container. Useful for accessing your application from the host.
- `-v /home/user/app-data:/app/data`: Mounts a volume from `/home/user/app-data` on the host to `/app/data` inside the container. This is useful for persistent or shared data.
- `-e SECRET_KEY=some_secret_value`: Sets an environment variable inside the container.
- `--network my-custom-network`: Connects the container to a specified network. Useful for inter-container communication.
- `node:16-alpine`: Specifies the image to use for the container. In this case, an Alpine-based Node.js 16 image.

**Important Caveats:**

- **Base Image:** If you're developing your application, you might have a `Dockerfile` that specifies how to build your own image. Replace `node:16-alpine` with the name of your image after building it.
- **Network:** The network `my-custom-network` must exist before you can connect a container to it. You can create a network using `docker network create my-custom-network`.

By understanding and using these commands, you'll be able to start leveraging Docker for your development and deployment workflows effectively.

## Introduction to Docker Compose

Docker Compose is particularly useful when you need to run multiple Docker containers. It allows you to define a multi-container application in a single file, then spin up your application with a single command (`docker compose up`) that starts all the services defined in your `docker-compose.yml` file.

Here's an example of a `docker-compose.yml` file for a simple web application:

## Docker Compose Example: Node.js Application with MongoDB

First, ensure you have Docker Compose installed. If not, follow the installation instructions on the [official Docker website](https://docs.docker.com/compose/install/).

### Step 1: Create a Dockerfile for Your Node.js Application

Let's assume you have a basic Node.js application. Create a `Dockerfile` in your application's root directory:

```Dockerfile:Dockerfile
# Use the official Node.js 16 as a parent image
FROM node:16-alpine

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install

# Bundle your app's source code inside the Docker image
COPY . .

# Make port 3000 available to the world outside this container
EXPOSE 3000

# Define the command to run your app
CMD [ "node", "app.js" ]
```

### Step 2: Define Your docker-compose.yml

Create a `docker-compose.yml` file in the root directory of your project. This file will define two services: one for your Node.js application and another for MongoDB:

```yaml:docker-compose.yml showLineNumbers
version: '3.8'
services:
  app:
    container_name: nodejs_app
    build: .
    ports:
      - "8080:3000"
    environment:
      - MONGO_DB_USERNAME=admin
      - MONGO_DB_PASSWORD=password
    depends_on:
      - mongo
    networks:
      - app-network

  mongo:
    container_name: mongo_db
    image: mongo:latest
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    volumes:
      - mongo-data:/data/db
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  mongo-data:
    driver: local
```

This `docker-compose.yml` file does the following:

- Defines two services: `app` (your Node.js application) and `mongo` (a MongoDB database).
- Builds the `app` service from the Dockerfile in the current directory.
- Maps port 8080 on the host to port 3000 on the container for the `app` service.
- Sets environment variables for both services. These are used for MongoDB authentication.
- Specifies a volume `mongo-data` for persisting data generated by and used by MongoDB.
- Defines a custom network `app-network` for communication between the containers.

### Step 3: Running Your Application with Docker Compose

Navigate to the directory containing your `docker-compose.yml` file and run:

```shell:terminal
docker compose up
```

This command builds the images (if they don't exist) and starts the containers as defined in your `docker-compose.yml` file. To run the containers in detached mode, use `docker compose up -d`.

To stop and remove all the containers defined in the `docker-compose.yml` file, use:

```shell:terminal
docker compose down
```

when you run docker compose up, it will start the containers as defined in your `docker-compose.yml` file. Beacuse you have build: . in your `docker-compose.yml` file, it will build the images if they don't exist. For that you need to have a Dockerfile in the root directory of your project.
