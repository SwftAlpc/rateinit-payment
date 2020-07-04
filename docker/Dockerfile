FROM golang:1.14.4-alpine3.12 as build

LABEL maintainer="https://github.com/SwftAlpc"

WORKDIR /tmp

ENV PROTOBUF_VERSION 3.12.3
ENV PROTOBUF_URL https://github.com/protocolbuffers/protobuf/releases/download/v$PROTOBUF_VERSION/protobuf-cpp-$PROTOBUF_VERSION.tar.gz

RUN set -eux && \
    apk update && \
    apk add --no-cache git curl build-base autoconf automake libtool && \
    curl -L -o /tmp/protobuf.tar.gz $PROTOBUF_URL && \
    tar xvzf protobuf.tar.gz

WORKDIR /tmp/protobuf-$PROTOBUF_VERSION

RUN set -eux && \
    ./autogen.sh && \
    ./configure && \
    make -j 3 && \
    make install && \
    go get -u github.com/golang/protobuf/protoc-gen-go && \
    go get -u github.com/pseudomuto/protoc-gen-doc/cmd/protoc-gen-doc

WORKDIR /go/app

COPY . .

RUN set -eux && \
    go build -o rateinit-payment && \
    curl -fLo /go/bin/air https://git.io/linux_air && \
    chmod +x /go/bin/air && \
    go get -u github.com/go-delve/delve/cmd/dlv && \
    go build -o /go/bin/dlv github.com/go-delve/delve/cmd/dlv

FROM alpine:3.12

WORKDIR /app

COPY --from=build /go/app/rateinit-payment .

RUN set -x && \
    addgroup go && \
    adduser -D -G go go && \
    chrown -R go:go /app/rateinit-payment

CMD ["./rateinit-payment"]