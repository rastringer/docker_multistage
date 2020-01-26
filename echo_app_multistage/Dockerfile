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