# Dockerized nextdns client
You can set NEXTDNS_ID environment variable to use your own Endpoint ID which can be found on the NextDNS Setup page. You can also pass custom command line parameters to nextdns-cli with NEXTDNS_ARGUMENTS environment variable.

## Compose example:
```
    version: "3"
    services:
      nextdns:
        image: bizkut/docker-nextdns
        container_name: nextdns
        environment:
          - NEXTDNS_ID=abcdef  # (optional) replace with your own Endpoint ID or remove this line
          - NEXTDNS_ARGUMENTS=-listen :5353 -report-client-info -log-queries -cache-size 10MB -max-ttl 5s  # custom CLI options
        ports:
          - 53:5353/udp
        restart: always
```
## Dockerfile
```
FROM golang:alpine AS builder

ENV CGO_ENABLED=0
ENV NEXTDNS_ARGUMENTS="-listen :5353 -report-client-info -log-queries -cache-size 10MB -max-ttl 5s"
ENV NEXTDNS_ID=abcdef
RUN set -xe && \
    apk add git upx && \
    git clone https://github.com/nextdns/nextdns.git /src && \
    cd /src && \
    go build -ldflags="-s -w" -o /go/bin/nextdns && \
    upx --lzma /go/bin/nextdns

FROM alpine
COPY --from=builder /go/bin/nextdns /usr/bin
EXPOSE 5353/udp
CMD /usr/bin/nextdns run ${NEXTDNS_ARGUMENTS} -config ${NEXTDNS_ID}
```
