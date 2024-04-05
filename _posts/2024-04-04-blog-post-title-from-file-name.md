## My scratchpad for useful docker commands
(because the documentation sometimes gets too irritating)

----
Dockerfile example:
```
# Sample dockerfile to illustrate how to setup a self-contained myprogram install. I'm using myprogram.contourer without GUI as a demonstration.
# 1) Install docker in the system
# 2) Create a folder with all files: this Dockerfile, the <filename>.tar.gz file
# 3) From this folder, run:
#       docker build --no-cache -t rocky9,myprogram:0.1 ./
# where the name 'rocky9,myprogram:0.1' is somewhat arbitrary and can be tinkered with
# 4) Fol#
# Sample dockerfile to illustrate how to setup a self-contained myprogram install. I'm using myprogram.contourer without GUI as a demonstration.
# 1) Install docker in the system
# 2) Create a folder with 3 files: this Dockerfile, the myprogram.contourer.tar.gz file and the Minuit2-master.zip file
# 3) From this folder, run:
#       docker build --no-cache -t rocky9,myprogram:0.1 ./
# where the name 'rocky9,myprogram:0.1' is somewhat arbitrary and can be tinkered with
# 4) Following it running fully, it will show up in the list of images that can be opened as
#       docker images --digests
# 5) Attach to this container thereafter by running 
#       docker run -it rocky9,myprogram:0.1
#   and exit from the session by 'exit'
#            -Sudarsan B, 16 Nov 2023, bsudarsan92-at-gmail.com
#
FROM rockylinux:9
LABEL maintainer=sud
LABEL version=0.1
COPY placeholderprogram.tar.gz /home/
RUN dnf upgrade -y --refresh && \
 dnf group install -y "Development Tools" && \
 dnf install -y readline-devel && \
 dnf update && \
 dnf install -y epel-release && \
 dnf -y install root && \
 dnf -y install nano 
WORKDIR /home/
ENV DOCKERHOME=.

RUN ls && \
 ls /home/ && \
 cd /home/ && \
 tar -xvf placeholderprogram.tar.gz && \
 mkdir /home/placeholderprogram/AprMay2024/ && \
 cd placeholderprogram && \
 make
CMD /bin/bash
```
Navigate to a directory containing a Dockerfile, and run the following:
```
sudo docker build --no-cache -t docker.containername:0.1 .
```
It is nice to have a yaml file that allows one to tailor and create any maps between local and container directories, for ex. Do that by saving the following as gs_loader.yaml (say)

```
services:
  rocky9.gs:
    # platform: linux/arm64,linux/amd64 
    build:
      context: .
#      additional_contexts:
#        - mysrc=${DOCKERHOME}/provisioning
#      dockerfile: ${DOCKERHOME}/DockerConfigs/GODDESS_SORT_Fedora.dockerfile
    environment:
      - DISPLAY=host.docker.internal:0
      - LANG=${LANG}
      - TERM=xterm-256color
    volumes:
      - /etc/localtime:/etc/localtime:ro
#      - ${DOCKERHOME}/AprMay2024:/home/placeholderprogram/AprMay2024
      - ./AprMay2024/:/home/placeholderprogram/AprMay2024
    stdin_open: true
    tty: true
```


The built docker container can be attached to by running the following:
```
sudo docker compose -f gs_loader.yaml run rocky9.gs
```

And you will be in the container, reading and writing to the host's AprMay2024 folder
