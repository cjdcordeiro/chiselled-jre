ARG MAVEN_IMAGE
ARG UBUNTU_RELEASE=22.04
ARG USER=app
ARG UID=101
ARG GROUP=app
ARG GID=101

FROM golang:1.18 AS chisel
ARG UBUNTU_RELEASE
RUN git clone https://github.com/canonical/chisel-releases /opt/chisel-releases \
    && git clone --depth 1 -b main https://github.com/canonical/chisel /opt/chisel
WORKDIR /opt/chisel
RUN go generate internal/deb/version.go \
    && go build ./cmd/chisel

FROM public.ecr.aws/ubuntu/ubuntu:$UBUNTU_RELEASE@sha256:7e421ec89eccbc7b0b38c0e4cea41548b4bd0d4cff63c42355fadd0369f49fa0 AS builder
SHELL ["/bin/bash", "-oeux", "pipefail", "-c"]
RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y ca-certificates \
        ca-certificates-java \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*
COPY --from=chisel /opt/chisel/chisel /usr/bin/

### BOILERPLATE END ###

FROM builder AS sliced-deps
ARG USER
ARG UID
ARG GROUP
ARG GID
SHELL ["/bin/bash", "-oeux", "pipefail", "-c"]
COPY --from=chisel /opt/chisel-releases /opt/chisel-releases
# use Java 8 headless AWT depedendencies

RUN mkdir -p /rootfs \
      && chisel cut --release /opt/chisel-releases --root /rootfs \
        openjdk-8-jre-headless_awt

FROM $MAVEN_IMAGE
COPY --from=sliced-deps /rootfs /
