## My scratchpad for useful docker commands
(because the documentation sometimes gets too irritating)

----
Dockerfile example:
```
# Sample dockerfile to illustrate how to setup a self-contained AZURE2 install. I'm using AZURE2.contourer without GUI as a demonstration.
# 1) Install docker in the system
# 2) Create a folder with all files: this Dockerfile, the <filename>.tar.gz file
# 3) From this folder, run:
#       docker build --no-cache -t rocky9.azure2:0.1 ./
# where the name 'rocky9.azure2:0.1' is somewhat arbitrary and can be tinkered with
# 4) Fol#
# Sample dockerfile to illustrate how to setup a self-contained AZURE2 install. I'm using AZURE2.contourer without GUI as a demonstration.
# 1) Install docker in the system
# 2) Create a folder with 3 files: this Dockerfile, the AZURE2.contourer.tar.gz file and the Minuit2-master.zip file
# 3) From this folder, run:
#       docker build --no-cache -t rocky9.azure2:0.1 ./
# where the name 'rocky9.azure2:0.1' is somewhat arbitrary and can be tinkered with
# 4) Following it running fully, it will show up in the list of images that can be opened as
#       docker images --digests
# 5) Attach to this container thereafter by running 
#       docker run -it rocky9.azure2:0.1
#   and exit from the session by 'exit'
#            -Sudarsan B, 16 Nov 2023, bsudarsan92-at-gmail.com
#
FROM rockylinux:9
LABEL maintainer=sud
LABEL version=0.1
COPY goddessSort.tar.gz /home/
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
 tar -xvf goddessSort.tar.gz && \
 mkdir /home/goddessSort/AprMay2024/ && \
 cd goddessSort && \
 make
CMD /bin/bash
```
Navigate to a directory containing a Dockerfile, and run the following:
```
sudo docker build --no-cache -t docker.containername:0.1 .
```
