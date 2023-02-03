# How to run a web application on Nginx in Docker
![Иллюстрация к проекту](https://github.com/konstantinnaumov/naumov_project/blob/main/Task%202/79ed10cd72b0067377f42b75002aa83b.jpg)

## Install Docker Engine on Ubuntu

### Prerequisites
#### OS requirements
To install Docker Engine, you need the 64-bit version of one of these Ubuntu versions:

Ubuntu Kinetic 22.10

Ubuntu Jammy 22.04 (LTS)

Ubuntu Focal 20.04 (LTS)

Ubuntu Bionic 18.04 (LTS)

Docker Engine is compatible with x86_64 (or amd64), armhf, arm64, and s390x architectures.
## Installation methods
Using a convenience scripts. Only recommended for testing and development environments.
## Install using the convenience script
Docker provides a convenience script at [https://get.docker.com/](https://get.docker.com/ "Convenience scripts") to install Docker into development environments non-interactively. The convenience script isn’t recommended for production environments, but it’s useful for creating a provisioning script tailored to your needs. Also refer to the install using the repository steps to learn about installation steps to install using the package repository. The source code for the script is open source, and can be found in the docker-install repository on GitHub.

Always examine scripts downloaded from the internet before running them locally. Before installing, make yourself familiar with potential risks and limitations of the convenience script:

The script requires root or sudo privileges to run.
The script attempts to detect your Linux distribution and version and configure your package management system for you.
The script doesn’t allow you to customize most installation parameters.
The script installs dependencies and recommendations without asking for confirmation. This may install a large number of packages, depending on the current configuration of your host machine.
By default, the script installs the latest stable release of Docker, containerd, and runc. When using this script to provision a machine, this may result in unexpected major version upgrades of Docker. Always test upgrades in a test environment before deploying to your production systems.
The script isn’t designed to upgrade an existing Docker installation. When using the script to update an existing installation, dependencies may not be updated to the expected version, resulting in outdated versions.
This example downloads the script from [https://get.docker.com/](https://get.docker.com/ "Convenience scripts") and runs it to install the latest stable release of Docker on Linux:

        curl -fsSL https://get.docker.com -o get-docker.sh
        sudo sh get-docker.sh

## Manage Docker as a non-root user
The Docker daemon binds to a Unix socket, not a TCP port. By default it’s the root user that owns the Unix socket, and other users can only access it using sudo. The Docker daemon always runs as the root user.

If you don’t want to preface the docker command with sudo, create a Unix group called docker and add users to it. When the Docker daemon starts, it creates a Unix socket accessible by members of the docker group. On some Linux distributions, the system automatically creates this group when installing Docker Engine using a package manager. In that case, there is no need for you to manually create the group.

### Warning
The docker group grants root-level privileges to the user. For details on how this impacts security in your system, see [Docker Daemon Attack Surface](https://docs.docker.com/engine/security/#docker-daemon-attack-surface ).
To create the docker group and add your user:

    sudo groupadd docker
    sudo usermod -aG docker $USER
You can also run the following command to activate the changes to groups:

    newgrp docker
Verify that you can run docker commands without sudo.

    docker run hello-world
This command downloads a test image and runs it in a container. When the container runs, it prints a message and exits.
## Install Angular CLI в Ubuntu 20.04 
### Installing Node.js
To manage the installed version of Node.js and make changes, you need the NVM (Node Version Manager) utility. It will be prioritized for working on the developer's local machine, for dev systems and testing new features in experimental versions. The ability to quickly change the version can be very useful in all of these cases. NVM on Ubuntu installs without problems. Let's proceed with the installation.

First, let's set the command to collect the list of packages from the repositories:

    sudo apt update

Some versions of Node.js need to be built, so it is important to install the required packages for this:

    sudo apt install build-essential checkinstall libssl-dev

To install or update nvm, you must run the [install script](https://github.com/nvm-sh/nvm/blob/v0.39.3/install.sh "install script"). To do this, you can download and run the script manually or use the following cURL command:

    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash

Running the above command downloads the script and runs it. The script clones the nvm repository into ~/.nvm and tries to add the source lines from the snippet below to the correct profile file ( ~/.bash_profile, ~/.zshrc, ~/.profile, or ~/.bashrc).
Troubleshooting Linux
On Linux, after running the install script, if you get nvm: command not found, or don't see a response from your terminal after typing command -v nvm, just close your current terminal, open a new terminal, and try again.

After all the commands executed, the nvm utility became available to us. Before installing Node.js, you need to familiarize yourself with the list of available versions:

    nvm ls-remote

The list is long, since Node.js has existed since 2010 and for some time even existed as two parallel projects, which after some time merged into one project again.
To get familiar with the Node.js server environment, you should choose LTS. It will be marked in the list with the corresponding comment.
You can install any version you need, for example, we need 18.13.0.

    nvm install 18.13.0

If you want to make some other version active, which you also previously installed, specify the command:

    nvm use 14.16.1

Let's check what we got:

    node -v
    npm -v


Before deleting a specific version, you must first deactivate it:

    nvm deactivate 14.16.1

And then delete with the command:

    nvm uninstall 14.16.1

It is no longer visible in the system.

    nvm list

### Installing the Angular CLI
After installing node.js and npm on your system, use the following commands to install the Angular cli tool on your system.

    npm install -g @angular/cli

The latest version of Angular CLI will be installed on your Ubuntu Linux system.
Using the -g command above will install the Angular CLI tool globally. Thus, it will be available to all users and applications in the system. Angular CLI provides the ng command used for command line operations. Let's check the installed version of ng on your system.

## Small preparations: creating a web application
First, we need some kind of web application that will be served by the Nginx server. You can use any JavaScript framework, our usage uses the Angular demo project:

    ng new angular-nginx-docker --minimal
    cd angular-nginx-docker
    ng build --configuration production

As a result of executing these commands, a new Angular application will be created, located in the `$PWD/dist/angular-nginx-docker` folder. This directory contains the `index.html` file and several JavaScript and CSS files that are typical for single page applications. The application, all code and commands from this article are in the [GitHub repository](https://github.com/konstantinnaumov/naumov_project/tree/main/Task%202).
## Run an Nginx application in a Docker container
Nginx and Docker work surprisingly well together. Instead of installing Nginx on our machine directly, we will run it in [Docker using nginx:alpine](https://hub.docker.com/r/yobasystems/alpine-nginx/) as an example.
The `nginx:alpine` image is very performant, although it only takes up about 20 MB of disk space. To run the `nginx:alpine` image interactively in a container, we run the following command:

    docker run -it -p 80:80 \
    -v /$PWD/dist/angular-nginx-docker:/usr/share/nginx/html:ro \
    nginx:alpine

The above command mounts the volume in the Docker container in `read only` mode (set using the flag: `ro`). The volume maps our application to the default Nginx web content root folder. The command also maps port `80` on the container to port `80` on the host so we can see our application on `http://localhost`. When the client requests the content of the application, we see the Nginx logs in the console.
Without additional configuration, Nginx uses default web server configuration files.
## Dockerizing a Web Application with Nginx
In the previous section, we learned how to work with our single page application using Nginx in a container. We can also ship our application, along with Nginx, as a Docker image to containers in other environments. Container applications provide a number of advantages: ease of updating, localization of failures, simple technology change. So now we will create a Docker image that contains both the web application and the Nginx server, which will automatically serve it when the Docker image is launched.
To get the Docker image, create the following `Dockerfile` in the existing `~/angular-nginx-docker` directory:

    # 1. Building an Angular App
    FROM node:alpine as builder
    
    WORKDIR /app
    COPY package.json package-lock.json ./
    ENV CI=1
    RUN npm ci
    
    COPY . .
    RUN npm run build --  --configuration production --output-path=/dist
    
    # 2. Deploying an Angular app on NGINX
    FROM nginx:alpine
    
    # Replacing the default nginx page with the corresponding web application
    RUN rm -rf /usr/share/nginx/html/*
    COPY --from=builder /dist /usr/share/nginx/html
    
    ENTRYPOINT ["nginx", "-g", "daemon off;"]

This `Dockerfile` includes two phases of creating a Docker image. It first pulls the `node:alpine` image to create an Angular app that is published to the `/dist` output path. Next, it switches to the `nginx:alpine` image, replacing the default Nginx application root with an Angula application. The last line specifies the image entry point for the command: `nginx -g daemon off;`. This ensures that Nginx stays "in the foreground" so that Docker can properly monitor the process (otherwise the container will stop immediately upon startup).

With the Dockerfile described above, we can create a Docker image and run it in a container with the restart function `--restart always` using the following commands:

    docker build -t angular-nginx-docker .
    docker run -d -p 80:80 --restart always angular-nginx-docker
    

    
