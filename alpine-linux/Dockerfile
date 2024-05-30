FROM alpine:latest
RUN apk update && apk add --no-cache openssh openrc
RUN rc-update add sshd && mkdir -p /run/openrc && touch /run/openrc/softlevel