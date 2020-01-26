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