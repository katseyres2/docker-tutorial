# Docker CLI

## docker port
```bash
docker port CONTAINER [PRIVATE_PORT[/PROTO]]
docker port test
docker port test 7890/tcp
docker port test 7890/udp
```

## docker ps
```bash
docker ps -a            # (--all)       show all containers
          -f            # (--filter)    filter output based on conditions provided
          --format      # pretty-print containers using a Go template
          -n            # (--last) show n last created containers (include all states)
          -l            # (--last) show n last created containers (includes all states)
          --no-trunc    # don't truncate output
          -q            # (--queit) only display container IDs
          -s            # (--size) display total file sizes
```

`--filter` has several key parameters : `id, name, label, exited, status, ancestor, before/since, volume, network, publish/expose, health, isolation, is-task`.  
It also has placeholders : `.ID, .Image, .Command, .CreatedAt, .RunningFor, .Ports, .State, .Status, .Size, .Names, .Labels, .Label, .Mounts, .Networks`. 

```bash
docker ps --filter ancestor=ubuntu
docker ps --format "{{.ID}}: {{.Command}}"
```

## docker exec
```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
    --detach, -d        # run the command in background
    --detach-keys       # override the key sequence for detaching a container
    --env, -e           # set environment variables
    --env-file          # read in a file of environment variables
    --interactive, -i   # keep stdin open even if not attached
    --privileged        # give extended privileges to the command
    --tty, -t           # allocate a pseudo-tty
    --user, -u          # username or uid
    --workdir, -w       # working directory inside the container
```
It only works if the container's primary process (PID 1) is running.

## docker inspect
```bash
docker inspect [OPTIONS] NAME|ID [NAME|ID...]
    --format, -f        # format the output using the given Go template
    --size, -s          # display total file sizes if the type is container
    --type              # return json for specified type
```

## docker build
```sh
docker build [OPTIONS] PATH | URL | -
    --add-host                  # add a custom host-to-ip mapping
    --build-arg                 # set build-time variables
    --cache-from                # images to consider as cache sources
    --cgroup-parent             # optional parent cgroup for the container
    --compress                  # compress the build context using gzip
    --cpu-period                # limit the cpu cfs (completely fair scheduler) period
    --cpu-quota                 # limit the cpu cfs quota
    --cpu-shares, -c            # cpu shares
    --cpuset-cpus               # cpus in which allow execution
    --cpuset-mems               # mems in which allow execution
    --disable-content-trust     # skip image verification
    --file, -f                  # name of the Dockerfile
    --force-rm                  # always remove intermediate containers
    --iidfile                   # write the image id to the file
    --isolation                 # container isolation technology (default, process, hyperv)
    --label                     # set metadata for an image
    --memory, -m                # memory limit
    --memory-swap               # swap limit equals to memory plus swap
    --network                   # set the networking mode for the RUN instruction during the build
    --no-cache                  # do not use cache when building the image
    --output, -o                # output destination
    --platform                  # set platform if server is multi-platform capable
    --progress                  # set type of progress output
    --pull                      # always attempt to pull a newer version of the image
    --quiet, -q                 # suppress the build output and print image ID on success
    --rm                        # remove intermediate containers after a successful build
    --secret                    # secret file to expose to the build
    --security-opt              # security options
    --shm-size                  # size of /dev/shm
    --squash                    # squash newly built layers into a single new layer
    --ssh                       # ssh agent socket or keys to expose
    --stream                    # stream attaches to server to negotiate build context
    --tag, -t                   # name and optionally a tag in the 'name:tag' format
    --target                    # set the target build stage to build
    --ulimit                    # ulimit options

docker build - < Dockerfile     # send Dockerfile in the URL
```
Build a Docker image from a Dockerfile and a context.  
The build context is the set of files located in `PATH` or `URL`. If `PATH` points to a git repository, it acts itself as the build context.

```sh
# syntax                    # commit                       # build context

myrepo.git                  # refs/heads/master             /
myrepo.git#mytag            # refs/tags/mytag               /
myrepo.git#mybranch         # refs/heads/mybranch           /
myrepo.git#pull/42/head     # refs/pull/42/head             /
myrepo.git#mytag:myfolder   # refs/tags/mytag               /myfolder
```

## docker run

```sh
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
    --add-host                  # add a custom host-to-ip mapping
    --attach, -a                # attach to stdin, stdout or stderr
    --blkio-weight              # block io
    --blkio-wieght-device       # block io weight
    --cap-add                   # add linux capabilities
    --cap-drop                  # drop linux capabilities
    --cgroup-parent             # optional cgroup parent for the container
    --cgroupns                  # ...
    --cidfile                   # write the container id to the file
    --cpu-count                 # cpu count (windows only)
    --cpu-percent               # cpu percent (windows only)
    --cpu-period                # limit cpu cfs period
    --cpu-quota                 # limit cpu cfs quota
    --cpu-rt-period             # limit cpu real-time period in microseconds
    --cpu-rt-runtime            # limit cpu real-time runtime in microseconds
    --cpu-shares, -c            # cpu shares
    --cpus                      # number of cpus
    --cpuset-cpus               # cpus in which to allow execution
    --cpuset-mems               # mems in which to allow execution
    --detach, -d                # run container in background and print container id
    --detach-keys               # override the key sequence for detaching a container
    --device                    # add a host device to the container
    --device-cgroup-rule        # add a rule to the cgroup allowed devices list
    --device-read-bps           # limit read rate (bytes/second) from a device
    --device-read-iops          # IO/second
    --device-write-bps          # limit write rate (bytes/second) from a device
    --device-write-iops         # IO/second
    --disable-content-trust     # skip image verification
    --dns                       # set custom dns servers
    --dns-opt                   # set dns options
    --dns-search                # set custom dns search domains
    --domainname                # container nis domain name
    --entrypoint                # overwrite the default ENTRYPOINT of the image
    --env, -e                   # set environment variables
    --env-file                  # reaad in a file of environment variables
    --expose                    # expose a port or a range of ports
    --gpus                      # gpu devices to add to the container
    --group-add                 # add additionnal groups to join
    --health-cmd                # command to run to check health
    --health-interval           # time between running the check
    --health-retries            # consecutive failres needed to report unhealthy
    --help                      # start period for the container ot initialize before starting health-restries countdown
    --hostname, -h              # container host name
    --init                      # run an init inside the container that forwards signals and reaps processes
    --interactive, -i           # keep stdin open even if not attached
    --io-maxbandwidth           # maximum IO bandwidth limit for the system drive (windows only)
    --io-maxiops                # maximum IOps limit for the system drive (windows only)
    --ip                        # IPv4 address
    --ip6                       # IPv6 address
    --ipc                       # IPC mode to use
    --isolation                 # container isolation technology
    --kernel-memory             # kernel memory limit
    --label, -l                 # set meta data on a container
    --label-file                # rad in a line delimited file of labels
    --link                      # add link to another container
    --link-local-ip             # container IPv4/IPv6 link-local addresses
    --log-driver                # logging driver for the container
    --log-opt                   # log driver options
    --mac-address               # container MAC address
    --memory, -m                # memory limit
    --memory-reservation        # memory soft limit
    --memory-swap               # swap limit equal to memory plus swap
    --memory-swapppiness        # tune container memory swappiness
    --mount                     # attach a filesystem mount to the container
    --name                      # assign a name to the container
    --net                       # connect a container to a network
    --net-alias                 # add network-scoped alias for the containre
    --no-healthcheck            # disable any container-specified HEALTHCHECK
    --oom-kill-disable          # disable OOM Killer
    --oom-score-adj             # turn host's OOM preferences
    --pid                       # PID namespace to use
    --pids-limit                # tune container pids limit
    --platform                  # set platform if server is multi-platform capable
    --privileged                # give extended privileges to this container
    --publish, -p               # publish a container's port(s) to the host
    --publish-all, -P           # publish all exposed ports to random ports
    --pull                      # pull image before running (always, missing, never)
    --read-only                 # mount the container's root filesystem as read only
    --restart                   # restart policy to apply when a container exits
    --rm                        # automatically remove the container when it exits
    --runtime                   # runtime to use for this container
    --security-opt              # security options
    --shm-size                  # size of /dev/shm
    --sig-proxy                 # proxy received signals to the process
    --stop-signal               # signal to stop a container
    --stop-timeout              # timeout (seconds) to stop a container
    --storage-opt               # storage driver options for the container
    --sysctl                    # sysctl options
    --tmpfs                     # mount a tmpfs directory
    --tty, -t                   # allocate a pseudo-TTY
    --ulimit                    # ulimit options
    --user, -u                  # username or uid
    --userns                    # user namespace to use
    --uts                       # UTS namespace to use
    --volume, -v                # bind mount a volume
    --volume-driver             # optional volume dirver for the container
    --volumes-from              # mount volumes from the specified container(s)
    --work-dir, -w              # working directory inside the container
```
