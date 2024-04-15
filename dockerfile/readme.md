# Docker

First of all, RTFM with
```bash
docker <command> --help
```

  * [Format](#format)
  * [Parser directives](#parser-directives)
  * [Environment](#environment)
  * [*dockerignore*](#dockerignore)
  * [FROM](#from)
  * [RUN](#run)
    + [RUN --mount](#run---mount)
    + [RUN --network](#run---network)
    + [RUN --security](#run---security)
  * [CMD](#cmd)
  * [LABEL](#label)
  * [EXPOSE](#expose)
  * [ENV](#env)
  * [Build](#build)
  * [Share](#share)
  * [Run](#run)
  * [Run](#run-1)
  * [Example](#example)

sources :
- introduction : https://docs.docker.com/storage/bind-mounts/
- mount : https://docs.docker.com/engine/reference/builder/#understand-how-arg-and-from-interact

---

## Format
The character `\` allow multiline instruction. Multiline commits are not supported.

```docker
# Comment               # the line is ignored
INSTRUCTION arguments

RUN echo hello \        
# comment               # this one too (echo hello world)
world

RUN echo hello \        # same as the previous instruction
world
```

## Parser directives
- don't support multiline
```docker
# direc \
tive=value
```
- must be unique
```docker
# directive=value1
# directive=value2

FROM ImageName
```
- must be above `FROM` instruction
```docker
FROM ImageName
# directive=value
```
- must not be below commit
```docker
# About my dockerfile
# directive=value
FROM ImageName
```
Examples :
```docker
# syntax=docker/dockerfile:1                              # update all minors and releases from version 1.x.x
# syntax=docker.io/docker/dockerfile:1.2                  # update all minors
# syntax=docker.io/docker/dockerfile:1.2.4                # immutable
# syntax=docker.io/docker/dockerfile:1.2.4-labs           # add -labs to get all new features in labs channel
# syntax=example.com/user/repo:tag@sha256:abcdef...

# escape=\ (backslash)
# escape=` (backtick)                                     # avoid ambiguity with powershell '\' division path
```

## Environment
`$variable_name` or `${variable_name}` is the same variable.  
`${variable:-word}` : if variable is not set, outputs 'world'.  
`${variable:+word}` : if variable is set, outputs 'world'.  

Environment variable is supported in these Dockerfile instructions : `ADD`, `COPY`, `ENV`, `EXPOSE`, `FROM`, `LABEL`, `STOPSIGNAL`, `USER`, `VOLUME`, `WORKDIR`, `ONEBUILD`.

Variable is keeping the same value a long the instructions.
```docker
# assign value 'hello' to the variable abc
ENV abc=hello
# assign new value to abc BUT def is still equals to 'hello'
ENV abc=bye def=$abc
# ghi equals to 'bye' because the above instruction updated abc variable
ENV ghi=$abc
```

## *dockerignore*
```docker
# excludes '/somedir/temporary.txt' and '/somedir/temp'
*/temp*
# excludes '/somedir/subdir/temporary.txt'
*/*/temp*
# excludes '/tempa' and '/tempb'
temp?

*.md          
# excludes all '.md' files excepted README.md
!README.md

*.md
!README*.md
# excludes all '.md' files excepted 'README*.md' which don't match with 'README-secret.md'
README-secret.md

*.md
# muted by the next instruction
README-secret.md
# excludes all '.md' files excepted all 'README*.md' files
!README*.md
```

## FROM
It has 3 forms :
- `FROM [--platform=<platform>] <image> [AS <name>]`
- `FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]`
- `FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]`

`ARG` is the only instruction that may precedes `FROM`.  
The `--platform` flag can specify the platform (`linux/amd64`, `linux/arm64`, `windows/amd64`).

Examples of `ARG` behaviours : 
```docker
ARG VERSION=latest
FROM busybox:$VERSION
# you must specify you want use $VERSION inside the build,
# otherwise it won't be accessible once the 'FROM' instruction gets executed
ARG VERSION                             

RUN echo $VERSION > image_version
```

## RUN
It has 2 forms :
- the shell form : `RUN <command>` (equals to `/bin/sh -c` or `cmd /S /C`)
- the exec form : `RUN ["executable", "param1", "param2"]`

If you want use a different shell like, you must use the second option.  
```docker
RUN ["/bin/bash", "-c", "echo hello"]
```
### RUN --mount
It has a multiple key-value pairs and their organization is not significant.  
`RUN --mount=[type=<type>][,option=<value>[,option=<value>]...]`

There are different kind of \<TYPE\>.  
- `type=bind` (default) : bind-mount context directories (read-only).  
```docker
RUN --mount=type=bind,target="/target_file",source="/source_file",from=<build>,rw
```

> __target :__ mount path  
  __source :__ source path in the FROM, default to the root of the FROM  
  __from :__ build stage or image name for the root of the source, default to the build context  
  __rw :__ read-write if set

- `type=cache` : mount a temporary directory for to cache directories for compilers and package managers. Cache mounts should only be used for better performance.
```docker
RUN --mount=type=cache,id=12244,target="/target_file"ro,sharing=shared,from=<build>,source="/source_file"mode=0755,uid=0,gid=0
```
> __id :__ optional id ot identify separate/different caches. default to value of target  
  __target :__ mount path  
  __ro :__ read-only if set  
  __sharing :__ sharing=shared  :  
  \- `default` : value, it can be used by multiple writers  
  \- `sharing=private` : create a new mount if there are multiple writers  
  \- `sharing=locked`  : pause the second writer until the first one releases the mount.  
  __from :__ build stage to use as a base of the cache mount, default to empty directory  
  __source :__ subpath in the FROM ot mount, default to the root of the FROM  
  __mode :__ file mode for new cache directory in octal, default is 0755  
  __uid :__ user id for new cache directory, default is 0  
  __gid :__ group id for new cache directory, default is 0

- `type=tmpfs` : allow mounting temporary file system in the build container.
```docker
RUN --mount=type=tmpfs,target="/target_file",size=1000000
```
>  __target :__ mount path  
   __size :__ specify an upper limit on the size of the filesystem

- `type=secret` : allow the build container to access secure files such a private keys without backing them into the image.  
```docker
RUN --mount=type=secret,id=1,target="/target_file",required=false,mode=0400,uid=0,gid=0
```
> __id :__ id of the secret, default to basename of the target path  
__target :__  mount path, default ot /run/secret/{id}  
__required :__ if set to true, the instruction errors out when the secret is unavailable, default to false  
__mode :__ file mode for secret file in octal, default 0400  
__uid :__ user id for secret file, default 0  
__gid :__ group id for secret file, default 0  
- `type=ssh` : allow the build container to access ssh keys via ssh agents, with support for passphrases.

```docker
RUN --mount=type=secret,id=1,target="/target_file",required=false,mode=0600,uid=0,gid=0					
```

> __id :__ id of ssh agent socket or key, default to "default"  
  __target :__ ssh agent socket path, default to /run/buildkit/ssh_agent.${N}  
  __required :__ if set to true, the instruction errors out when the secret is unavailable, default to false  
  __mode :__ file mode for secret file in octal, default 0600  
  __uid :__ user id for socket, default 0  
  __gid :__ group id for socket, default 0

*PS : you can use -v or --volume if you're controlling the --mount flag.*

### RUN --network
```docker
RUN --network=<type>
```

There are 3 \<TYPES> of network :
- `default` (default) : run in the default network, equivalent to not supplying a flag at all, the command is run in the default network for the build.
- `none` : run with no network access, `lo` is still available but it is isolated to this process.
- `host` : run in the host's network environment

### RUN --security
```docker
RUN --security=<type>
```

There are 2 types of security :
- `unsecure` : allow to run flows requiring elevated privileges, equivalent to `docker run --privileged`.
- `sandbox`

## CMD
It can only be one `CMD` instruction in the Dockerfile. If you list more than one, **the last one** will take effect.  
It provides defaults for an executing container.  

There are 3 forms : 
- exec form : `CMD ["executable","param1","param2"]`
- (as default parameter to ENTRYPOINT) : `CMD ["param1","param2"]`
- shell form : `CMD command param1 param2`

## LABEL
```dockerfile
LABEL <key>=<value>
```

Add metadata to an image.  
An image can have more than one LABEL.  
LABELs included in the base or the parent images (via FROM) are inherited by your image.  
To view image's labels : `docker image inspect` (add `--format` to show only LABELs)

## EXPOSE
You can specify if the container is listening on the port with the protocol udp or tcp.  
To expose port :
- expose manually : `docker run -p 80:80/tcp -p 80:80/udp`
- publish all exposed port in the Dockerfile : `docker run -P`

```dockerfile
EXPOSE <port> [<port>/<protocol>...]

EXPOSE 443
EXPOSE 80/udp
EXPOSE 80/tcp
```

## ENV
The value will be in the environment for all subsequent instruction in the build stage.
```dockerfile
ENV <key>=<value> ...

ENV MY_NAME="John Doe"
ENV MY_DOG=Rex\ The\ Dog
ENV MY_CAT=fluffy
```

To view ENV values : `docker inspect`  
To change ENV value on build image : `docker run <key>=<value>`


## Build
```bash
docker build -t myimage:1:0 .	# build an image form the Dockerfile in the current directory and tag the image
docker image ls					# list all images that are locally stored with the Docker engine
docker image rm alpine:3.4		# delete an image from the local image store
```

## Share
```bash
docker pull myimage:1.0						# pull an image from a registry
docker tag myimage:1.0 myrepo/myimage:2.0	# retag a local image with a new image name and tag
docker push myrepo/myimage:2.0				# push an image to a registry
```

## Run
```bash
docker container run --name web -p 5000:80 alpine:3.9	# run a container from the alpine version 3.9 image,
														# naming the running container "web" and expose port 5000 externally,
														# mapped to port 80 inside the container

docker container stop web									# stop a running container through SIGTERM
docker container kill web									# stop a running container through SIGKILL
docker network ls											# list the networks
docker container ls											# list the running containers (add --all to include stopped containers)
docker container rm -f $(docker ps -aq)						# delete all running and stopped containers
docker container logs --tail 100 web						# print the last 100 lines of a container-s logs
docker start web											# start the existing container
docker run -itd myimage /bin/bash							# -i keeps the STDIN open even if not attached
															# -t allocates a pseudo-TTY
															# -d runs container in backgoround and print container ID
docker exec [--name <username>] -it mycontainer /bin/bash	# execute container and launch bash CLI
```

## Run
```bash
sudo docker run -it alpine
```

## Example
```bash
docker run -dp 80:80 docker/getting-started

wget https://github.com/docker/getting-started/archive/refs/heads/master.zip
unzip master.zip
cd getting-started/app

docker build -t getting-started .
docker run -dp 3000:3000 getting-started
docker ps -a

docker stop -f <container_id>
docker rm <container_id>

docker push katseyres/getting-started:tagname
docker login -u YOUR-USER-NAME
docker tag getting-started YOUR-USER-NAME/getting-started

docker run -d ubuntu bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"
docker exec <container-id> cat /data.txt
docker run -it ubuntu ls /
```