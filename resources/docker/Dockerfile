FROM golang:1.22 AS build

ENV CGO_ENABLED=0
ENV GOOS=linux
RUN useradd -u 10001 connect

WORKDIR /go/src/github.com/redpanda-data/connect/
# Update dependencies: On unchanged dependencies, cached layer will be reused
COPY go.* /go/src/github.com/redpanda-data/connect/
RUN go mod download

# Build
COPY . /go/src/github.com/redpanda-data/connect/
# Tag timetzdata required for busybox base image:
# https://github.com/benthosdev/benthos/issues/897
RUN make TAGS="timetzdata"

# Pack
FROM oven/bun:debian

RUN pipx install poetry

LABEL maintainer="lunacrafts <lunacrafts@protonmail.com>"
LABEL org.opencontainers.image.source="https://github.com/mirrorboards-forks/connect"

WORKDIR /

RUN apt update
RUN apt install pipx
RUN pipx ensurepath
RUN pipx ensurepath --global
RUN pipx install poetry

COPY --from=build /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=build /etc/passwd /etc/passwd
COPY --from=build /go/src/github.com/redpanda-data/connect/target/bin/redpanda-connect .
COPY ./config/docker.yaml /connect.yaml

USER connect

EXPOSE 4195

ENTRYPOINT ["/redpanda-connect"]

CMD ["-c", "/connect.yaml"]
