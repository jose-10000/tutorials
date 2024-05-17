# Docker for Beginners


## What is Docker?
Docker is a tool designed to make it easier to create, deploy, and run applications by using containers. 

### What are containers?

Containers allow a developer to package an application with all the necessary parts, such as libraries and other dependencies, and ship it as a single package. As a result, the container gives the developer the confidence that the application will run on any other Linux machine, regardless of any custom settings that the target machine may have that differ from the machine on which the code was written and tested.

## Why containers?
When you create an application and want to share it, the typical scenario is "it works on my PC, why not on yours?
There are several reasons for this:
Our application runs on our PCs because we have the exact version of the dependencies it needs to do the job.
It is very difficult that we have two identical PCs, although they have the same software, hardly have the same versions of these, this generates compatibility problems and our app is affected.
The solution to this problem is to use containers, which are isolated environments that contain everything our application needs to run, so we can share our application with anyone and it will work the same way on any machine.
Thanks to a container, we do not need to install any of the applications or dependencies that our application will require on the target PC.

## Ok, now let's see how to use Docker

Essentially, Docker has two concepts that we should be familiar with.: the Dockerfile and Docker images.
The Dockerfile is a file that contains the instructions to build a Docker image, and the Docker image is a file(aka template) that contains the application and all its dependencies.

In other words, a Docker file is used to create a Docker image, which is then used as a template for the creation of one or more Docker containers.

- Example of a Dockerfile
```Dockerfile
FROM python:3.8-slim-buster                 # Base image, in this case, python 3.8 an is pulled from docker hub, it is an images repository,(https://hub.docker.com/_/python)

WORKDIR /app                                # Set the working directory to /app

COPY requirements.txt requirements.txt      # Copy the requirements.txt file to the working directory
RUN pip3 install -r requirements.txt        # Install the dependencies

COPY . .                                    # Copy the current directory contents into the container at /app, the current directory is the directory where the Dockerfile is located

CMD [ "python3", "-m" , "flask", "run", "--host=10.0.0.1"]  # Run the command when the container starts
```
