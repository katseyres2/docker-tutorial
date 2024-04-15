# syntax=docker/dockerfile:1
# escape=\

ARG version=latest
FROM alpine
# ARG version

# # abc == hello
# ENV abc=hello
# # abc == bye && def == hello
# ENV abc=bye def=$abc
# # ghi == bye
# ENV ghi=$abc

# shell form
# RUN echo ${VERSION} > image_version
# RUN ["/bin/sh", "-c", "echo hello > say_hello"]

# RUN apk add --no-cache openssh-client
# RUN mkdir -p -m ~/.ssh && ssh-keyscan gitlab.com >> ~/.ssh/kwown_hosts
# RUN --mount=type=ssh ssh -q -T

LABEL hello=world
LABEL username=ubuntu
LABEL version=6.6.6

EXPOSE 443
EXPOSE 80/udp
EXPOSE 80/tcp