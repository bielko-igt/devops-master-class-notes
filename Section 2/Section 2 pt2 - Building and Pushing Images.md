# Alpine

Alpine is a lightweight distribution of Linux ideal for small scale Docker images.

Python image using this distro: **python:alpine3.10**


# Dockerfile

## Base image (FROM ...)

The base image is an existing docker image from which our custom image gets built.

## Example Dockerfile

```dockerfile
FROM python:alpine3.10
WORKDIR /app 
COPY . /app
RUN pip install -r requirements.txt
EXPOSE 5000
CMD python ./launch.py
```
- created from base image python:alpine3.10 - python and pip preinstalled
- working directory for the application (within the virtual storage of the container)
- the python code and requirements.txt, located in the same directory as the Dockerfile ("." - current dir.), are copied to /app in the container, the same directory which will be used as the working directory
- the requirements are installed via pip
- port 5000 is exposed (for us to connect to from outside the container - 5000:5000)
- container launches python and runs the code

## Creating an image from a Dockerfile and pushing to Docker Hub

1. In your terminal, cd into the folder where Dockerfile is located
2. ```sh
   docker build -t in28min/hello-world-python:0.0.2.RELEASE <build context (current directory / .)>
   ```
   - this builds an image with the given tag from the Dockerfile in the current directory


Re-tag an image with a new tag, the new tag is then visible alongside the original image in "docker images"

```sh
docker tag <original tag> <new tag>
```
  
Log in to Docker Hub:
```sh
docker login
```
  - requests your Docker Hub dockerid and password
  
Afterwards, push to Docker Hub:
```sh
docker push <tag>
```
- tag has to be prefixed with your dockerid as the repository, i.e. "ladislavigt/test-image:0.0.1"

## Multi-stage builds

We may want to create an image via multiple separate stages.

The following is an example Dockerfile for building and running a Java application as a two-stage build:

```dockerfile
# Build a JAR File
FROM maven:3.8.2-jdk-8-slim AS stage1
WORKDIR /home/app
COPY . /home/app/
RUN mvn -f /home/app/pom.xml clean package

# Create an Image
FROM openjdk:8-jdk-alpine
EXPOSE 5000
COPY --from=stage1 /home/app/target/hello-world-java.jar hello-world-java.jar
ENTRYPOINT ["sh", "-c", "java -jar /hello-world-java.jar"]
```
 - the maven base image lets us build the project as a .jar file with dependencies
 - the jdk base image lets us run the built .jar application with Java
 - the following **instruction** is what links the two stages:
   ```dockerfile
   COPY --from=stage1 /home/app/target/hello-world-java.jar hello-world-java.jar
   ```
   - copies the built .jar file from the first stage (named stage1) into the current stage
 - the **ENTRYPOINT** instruction is preferred over **CMD** when specifying the executable/application file to run:
   ```dockerfile
   ENTRYPOINT ["sh", "-c", "java -jar /hello-world-java.jar"]
   ```
- the **RUN** instruction in **stage1** executes on top of the current image in a new layer. **Then** the result is committed to the image
   ```dockerfile
   RUN mvn -f /home/app/pom.xml clean package
   ```
   - the difference from ENTRYPOINT or CMD is that those commands don't commit the result as a new layer on the image

# Building Efficient Docker Images

Separating the dependencies and code into different layers on the image:

Before
 ```dockerfile
FROM node:8.16.1-alpine
WORKDIR /app
COPY . /app # <----
RUN npm install
EXPOSE 5000
CMD node index.js
```

After
 ```dockerfile
FROM node:8.16.1-alpine
WORKDIR /app
COPY package.json /app # <----
RUN npm install
EXPOSE 5000
COPY . /app # <----
CMD node index.js
```
- this separates dependencies from the code -> allows for dependencies in package.json to cache on their own layer
- that way, if you make changes to index.js, it won't have to re-run "npm install" because something in /app changed, but can cache from a previous run of this step, as index.js gets interacted with only at the last COPY now
- then, only the last EXPOSE, COPY, and CMD instructions will need to be re-run, allowing for shorter build times **while you are actively working on the application**

