<h1 align="center">Docker/도커</h1>
<p align="center"><img src="https://user-images.githubusercontent.com/86287920/191907717-0e0f13f8-56c3-4e49-b796-02b429a39f56.png" width="150"></p>

### Docker Overview
Docker is an open platform for developing, shipping, and running applications. Docker enables you to separate your applications from your infrastructure so you can deliver software quickly. With Docker, you can manage your infrastructure in the same ways you manage your applications. By taking advantage of Docker’s methodologies for shipping, testing, and deploying code quickly, you can significantly reduce the delay between writing code and running it in production.

### Docker architecture
Docker uses a client-server architecture. The Docker client talks to the Docker daemon, which does the heavy lifting of building, running, and distributing your Docker containers. The Docker client and daemon can run on the same system, or you can connect a Docker client to a remote Docker daemon. The Docker client and daemon communicate using a REST API, over UNIX sockets or a network interface. Another Docker client is Docker Compose, that lets you work with applications consisting of a set of containers.
<p align="center"><img src="https://docs.docker.com/engine/images/architecture.svg" width="800"></p>

### The Docker daemon
The Docker daemon (```dockerd```) listens for Docker API requests and manages Docker objects such as images, containers, networks, and volumes. A daemon can also communicate with other daemons to manage Docker services.

### Docker objects
When you use Docker, you are creating and using images, containers, networks, volumes, plugins, and other objects. This section is a brief overview of some of those objects.

#### Images
An *image* is a read-only template with instructions for creating a Docker container. Often, an image is based on another image, with some additional customization. For example, you may build an image which is based on the ```ubuntu``` image, but installs the Apache web server and your application, as well as the configuration details needed to make your application run.

You might create your own images or you might only use those created by others and published in a registry. To build your own image, you create a *Dockerfile* with a simple syntax for defining the steps needed to create the image and run it. Each instruction in a Dockerfile creates a layer in the image. When you change the Dockerfile and rebuild the image, only those layers which have changed are rebuilt. This is part of what makes images so lightweight, small, and fast, when compared to other virtualization technologies.

#### Containers
A container is a runnable instance of an image. You can create, start, stop, move, or delete a container using the Docker API or CLI. You can connect a container to one or more networks, attach storage to it, or even create a new image based on its current state.

By default, a container is relatively well isolated from other containers and its host machine. You can control how isolated a container’s network, storage, or other underlying subsystems are from other containers or from the host machine.

A container is defined by its image as well as any configuration options you provide to it when you create or start it. When a container is removed, any changes to its state that are not stored in persistent storage disappear.

#### Example ```docker run``` command
The following command runs an ```ubuntu``` container, attaches interactively to your local command-line session, and runs ```/bin/bash```.
```
$ docker run -i -t ubuntu /bin/bash
```

### Dockerfile reference
Docker can build images automatically by reading the instructions from a ```Dockerfile```. A ```Dockerfile``` is a text document that contains all the commands a user could call on the command line to assemble an image. Using ```docker build``` users can create an automated build that executes several command-line instructions in succession.

This page describes the commands you can use in a ```Dockerfile```. When you are done reading this page, refer to the ```Dockerfile``` Best Practices for a tip-oriented guide.

### Usage
The docker build command builds an image from a ```Dockerfile``` and a context. The build’s context is the set of files at a specified location ```PATH``` or ```URL```. The ```PATH``` is a directory on your local filesystem. The ```URL``` is a Git repository location.

The build context is processed recursively. So, a ```PATH``` includes any subdirectories and the ```URL``` includes the repository and its submodules. This example shows a build command that uses the current directory (```.```) as build context:
```
$ docker build .
```
The build is run by the Docker daemon, not by the CLI. The first thing a build process does is send the entire context (recursively) to the daemon. In most cases, it’s best to start with an empty directory as context and keep your Dockerfile in that directory. Add only the files needed for building the Dockerfile.
```
! WARNING
Do not use your root directory, /, as the PATH for your build context, as it causes the build to transfer the entire
contents of your hard drive to the Docker daemon.
```
## Best practices for writing Dockerfiles
This document covers recommended best practices and methods for building efficient images.

Docker builds images automatically by reading the instructions from a ```Dockerfile``` -- a text file that contains all commands, in order, needed to build a given image.

A Docker image consists of read-only layers each of which represents a Dockerfile instruction. The layers are stacked and each one is a delta of the changes from the previous layer. Consider this ```Dockerfile```:
```
FROM ubuntu:18.04
COPY . /app
RUN make /app
CMD python /app/app.py
```
Each instruction creates one layer:

- ```FROM``` creates a layer from the ```ubuntu:18.04``` Docker image.
- ```COPY``` adds files from your Docker client’s current directory.
- ```RUN``` builds your application with ```make```.
- ```CMD``` specifies what command to run within the container.

## Dockerfile instructions
These recommendations are designed to help you create an efficient and maintainable ```Dockerfile```.

### FROM
<a href="https://docs.docker.com/engine/reference/builder/#from">Dockerfile reference for the FROM instruction</a>
Whenever possible, use current official images as the basis for your images. We recommend the <a href="https://hub.docker.com/_/alpine/">Alpine image</a> as it is tightly controlled and small in size (currently under 6 MB), while still being a full Linux distribution.

### LABEL
<a href="https://docs.docker.com/config/labels-custom-metadata/">Understanding object labels</a>
You can add labels to your image to help organize images by project, record licensing information, to aid in automation, or for other reasons. For each label, add a line beginning with ```LABEL``` and with one or more key-value pairs. The following examples show the different acceptable formats. Explanatory comments are included inline.
```
Strings with spaces must be quoted or the spaces must be escaped. Inner quote characters ("), must also be escaped.
```
```
LABEL com.example.version="0.0.1-beta"
LABEL vendor1="ACME Incorporated"
LABEL vendor2=ZENITH\ Incorporated
LABEL com.example.release-date="2015-02-12"
LABEL com.example.version.is-production=""
```
An image can have more than one label. Prior to Docker 1.10, it was recommended to combine all labels into a single ```LABEL``` instruction, to prevent extra layers from being created. This is no longer necessary, but combining labels is still supported.
```
LABEL vendor=ACME\ Incorporated \
      com.example.is-beta= \
      com.example.is-production="" \
      com.example.version="0.0.1-beta" \
      com.example.release-date="2015-02-12"
```
See <a href="https://docs.docker.com/config/labels-custom-metadata/">Understanding object labels</a> for guidelines about acceptable label keys and values. For information about querying labels, refer to the items related to filtering in <a href="https://docs.docker.com/config/labels-custom-metadata/#manage-labels-on-objects">Managing labels on objects</a>. See also <a href="https://docs.docker.com/engine/reference/builder/#label">LABEL</a> in the Dockerfile reference.

### RUN
<a href="https://docs.docker.com/engine/reference/builder/#run">Dockerfile reference for the RUN instruction</a>
Split long or complex ```RUN``` statements on multiple lines separated with backslashes to make your ```Dockerfile``` more readable, understandable, and maintainable.

#### apt-get
Probably the most common use-case for ```RUN``` is an application of ```apt-get```. Because it installs packages, the ```RUN apt-get``` command has several gotchas to look out for.
Always combine ```RUN apt-get update``` with ```apt-get install``` in the same ```RUN``` statement. For example:
```
RUN apt-get update && apt-get install -y \
    package-bar \
    package-baz \
    package-foo  \
    && rm -rf /var/lib/apt/lists/*
```

### CMD
<a href="https://docs.docker.com/engine/reference/builder/#cmd">Dockerfile refrence for the CMD instruction</a>
The ```CMD``` instruction should be used to run the software contained in your image, along with any arguments. ```CMD``` should almost always be used in the form of ```CMD ["executable", "param1", "param2"…]```. Thus, if the image is for a service, such as Apache and Rails, you would run something like ```CMD ["apache2","-DFOREGROUND"]```. Indeed, this form of the instruction is recommended for any service-based image.

In most other cases, ```CMD``` should be given an interactive shell, such as bash, python and perl. For example, ```CMD ["perl", "-de0"]```, ```CMD ["python"]```, or ```CMD ["php", "-a"]```. Using this form means that when you execute something like ```docker run -it python```, you’ll get dropped into a usable shell, ready to go. ```CMD``` should rarely be used in the manner of ```CMD ["param", "param"]``` in conjunction with ```ENTRYPOINT```, unless you and your expected users are already quite familiar with how ```ENTRYPOINT``` works.

### EXPOSE
<a href="https://docs.docker.com/engine/reference/builder/#expose">Dockerfile reference for the EXPOSE instruction</a>
The ```EXPOSE``` instruction indicates the ports on which a container listens for connections. Consequently, you should use the common, traditional port for your application. For example, an image containing the Apache web server would use ```EXPOSE 80```, while an image containing MongoDB would use ```EXPOSE 27017``` and so on.

For external access, your users can execute ```docker run``` with a flag indicating how to map the specified port to the port of their choice. For container linking, Docker provides environment variables for the path from the recipient container back to the source (ie, ```MYSQL_PORT_3306_TCP```).

### ADD or COPY
- <a href="https://docs.docker.com/engine/reference/builder/#add">Dockerfile reference for the ADD instruction</a>
- <a href="https://docs.docker.com/engine/reference/builder/#copy">Dockerfile reference for the COPY instruction</a>
Although ```ADD``` and ```COPY``` are functionally similar, generally speaking, ```COPY``` is preferred. That’s because it’s more transparent than ```ADD```. ```COPY``` only supports the basic copying of local files into the container, while ```ADD``` has some features (like local-only tar extraction and remote URL support) that are not immediately obvious. Consequently, the best use for ```ADD``` is local tar file auto-extraction into the image, as in ```ADD rootfs.tar.xz /```.

If you have multiple ```Dockerfile``` steps that use different files from your context, ```COPY``` them individually, rather than all at once. This ensures that each step’s build cache is only invalidated (forcing the step to be re-run) if the specifically required files change.
For example:
```
COPY requirements.txt /tmp/
RUN pip install --requirement /tmp/requirements.txt
COPY . /tmp/
```
Results in fewer cache invalidations for the ```RUN``` step, than if you put the ```COPY . /tmp/``` before it.

Because image size matters, using ```ADD``` to fetch packages from remote URLs is strongly discouraged; you should use ```curl``` or ```wget``` instead. That way you can delete the files you no longer need after they’ve been extracted and you don’t have to add another layer in your image. For example, you should avoid doing things like:
```
ADD https://example.com/big.tar.xz /usr/src/things/
RUN tar -xJf /usr/src/things/big.tar.xz -C /usr/src/things
RUN make -C /usr/src/things all
```
And instead, do something like:
```
RUN mkdir -p /usr/src/things \
    && curl -SL https://example.com/big.tar.xz \
    | tar -xJC /usr/src/things \
    && make -C /usr/src/things all
```
For other items (files, directories) that do not require ```ADD```'s tar auto-extraction capability, you should always use ```COPY```.

### ENTRYPOINT
<a href="https://docs.docker.com/engine/reference/builder/#entrypoint">Dockerfile reference for the ENTRYPOINT instruction</a>
The best use for ```ENTRYPOINT``` is to set the image’s main command, allowing that image to be run as though it was that command (and then use ```CMD``` as the default flags).
for example:
```
ENTRYPOINT ["s3cmd"]
CMD ["--help"]
```
# Docker Project
This demo app shows a simple user profile app set up using 
- index.html with pure js and css styles
- nodejs backend with express module
- mongodb for data storage

All components are docker-based

### With Docker

#### To start the application

Step 1: Create docker network

    docker network create mongo-network 

Step 2: start mongodb 

    docker run -d -p 27017:27017 -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=password --name mongodb --net mongo-network mongo    

Step 3: start mongo-express
    
    docker run -d -p 8081:8081 -e ME_CONFIG_MONGODB_ADMINUSERNAME=admin -e ME_CONFIG_MONGODB_ADMINPASSWORD=password --net mongo-network --name mongo-express -e ME_CONFIG_MONGODB_SERVER=mongodb mongo-express   

_NOTE: creating docker-network in optional. You can start both containers in a default network. In this case, just emit `--net` flag in `docker run` command_

Step 4: open mongo-express from browser

    http://localhost:8081

Step 5: create `user-account` _db_ and `users` _collection_ in mongo-express

Step 6: Start your nodejs application locally - go to `app` directory of project 

    npm install 
    node server.js
    
Step 7: Access you nodejs application UI from browser

    http://localhost:3000

### With Docker Compose

#### To start the application

Step 1: start mongodb and mongo-express

    docker-compose -f docker-compose.yaml up
    
_You can access the mongo-express under localhost:8080 from your browser_
    
Step 2: in mongo-express UI - create a new database "my-db"

Step 3: in mongo-express UI - create a new collection "users" in the database "my-db"       
    
Step 4: start node server 

    npm install
    node server.js
    
Step 5: access the nodejs application from browser 

    http://localhost:3000

#### To build a docker image from the application

    docker build -t my-app:1.0 .       
    
The dot "." at the end of the command denotes location of the Dockerfile.
