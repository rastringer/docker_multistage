# docker_multistage
Example of multistage builds in Docker using a simple Go application.

Docker introduced the concept of multistage builds to remedy the problem of unnecessary development tools taking up space in an image, increasing its size and rendering deployments lengthier.

In this repo I give an example of a direct comparison of two identical applications, one with a single-stage and one a multi-stage Dockerfile. Our 'echo' app is a simple Go web server that takes a noun entered into the URL and tells us we love it (eg...localhost:8080/alpacas = "I love alpacas!"). In the first folder, ```echo_app``` has a Dockerfile that looks like this:


```
# First step: Build
FROM golang:1.13-alpine AS build 

WORKDIR /go/src/github.com/rastringer/echo_app 

# Copy all resources 
COPY . .

# Variables for build script 
ENV VERBOSE=0 
ENV PKG=github.com/rastringer/echo_app 
ENV ARCH=amd64 
ENV VERSION=test 

# # Run the build 
RUN go get -d -v ./...
RUN go install -v ./...

CMD ["echo_app"]
```

This Dockerfile builds the environment (Alpine, a stripped down Linux OS popular for containers due to its size) and compiles the code in one swoop. Here's the Dockerfile in the second folder, echo_app_multistage:

```
# First step: Build
FROM golang:1.13-alpine AS build 

WORKDIR /go/src/github.com/rastringer/echo_app_multistage 

# Copy all resources 
COPY . .

# Variables for build script 
ENV VERBOSE=0 
ENV PKG=github.com/rastringer/echo_app_multistage 
ENV ARCH=amd64 
ENV VERSION=test 

# # Run the build 
RUN go get -d -v ./...
RUN go install -v ./...

# Second step: deployment 
FROM alpine 

USER nobody:nobody 
COPY --from=build /go/bin/echo_app_multistage /bin/echo_app

CMD ["echo_app"]
```

We see here how the container OS is code compilation is done in the first build, and the deployment in a second stage. This means Docker first creates a 'build' image, before a second 'deployment' image, which comprises a vastly reduced binary. 

To run these files, clone this repo, ```cd``` into the respective folder, and run

```
docker build -t echo_app
```

and of course ```docker build -t echo_app_multistage``` for the second example. The '-t' in these commands adds the app name as a 'tag' which can be used to search and monitor (and more) your containers. Now we can compare our Docker images.

```
docker ps -a
```

will list all containers on your system, copy the container id from the terminal and run

```
docker image inspect [container id]
```

This command will list a reel of information in json. Here are the pertinent from my terminal:


Single-stage:
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 366716192,
        "VirtualSize": 366716192,

Multi-stage:
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 13041939,
        "VirtualSize": 13041939,
        
The multi-stage container, at 13041939 bytes, is 96% smaller than the single-stage.
