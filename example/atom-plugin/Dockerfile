# Stage 1: Build the plugin
FROM golang:1.23-alpine AS builder

# Set the working directory
WORKDIR /volcano

# Copy the source code into the container
RUN mkdir -p example/atom-plugin
#COPY example/atom-plugin /volcano/example/atom-plugin
COPY . /volcano

# Install necessary dependencies
RUN apk add --no-cache musl-dev gcc

# Build the plugin
#RUN CC=/usr/bin/x86_64-alpine-linux-musl-gcc CGO_ENABLED=1 go build -buildmode=plugin -o example/atom-plugin/atom.so example/atom-plugin/atom.go

# Stage 2: Create the runtime image
#FROM docker.io/volcanosh/vc-scheduler:latest
FROM golang:1.23 AS builder1
WORKDIR /volcano
COPY . /volcano

RUN wget http://musl.libc.org/releases/musl-1.2.1.tar.gz &&\
    tar -xf musl-1.2.1.tar.gz && cd musl-1.2.1 &&\
    ./configure &&\
    make && make install

WORKDIR /volcano
RUN CC=/usr/local/musl/bin/musl-gcc CGO_ENABLED=1 go build -buildmode=plugin -o example/atom-plugin/atom.so example/atom-plugin/atom.go
RUN CC=/usr/local/musl/bin/musl-gcc SUPPORT_PLUGINS="yes" make vc-scheduler



FROM alpine:latest
COPY --from=builder1 /volcano/_output/bin/vc-scheduler /vc-scheduler


# Create the plugins directory
RUN mkdir -p /plugins

# Copy the built plugin from the builder stage
COPY --from=builder1 /volcano/example/atom-plugin/atom.so /plugins/atom.so
WORKDIR /volcano
ENTRYPOINT ["/vc-scheduler"]
# Set the entrypoint or command if needed
# CMD ["your", "command", "here"]