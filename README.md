# pepperell lab
## What we are doing
* Install Docker
* Build a image with python, perl, samtools, popoolation
* Requirement for samtools install:
    - make
    - gcc
    - bash
    - perl
    - zlib
    - libbz2
    - liblzma
    - libcurl
    - libcrypto
    - reference: https://github.com/samtools/htslib/blob/develop/INSTALL
* Requirement for PoPoolation2 install:
    - Perl 5.8 or higher
    - R 2.7 or higher
    - a short read aligner (e.g.: bwa)
    - samtools
    - Mauve (http://asap.ahabs.wisc.edu/mauve/)
    - reference: https://sourceforge.net/p/popoolation/wiki/Manual/

## First, create your own Dockerhub account

https://hub.docker.com/
remember your username and password


## Then install Docker on your own machine

https://docs.docker.com/engine/install/
check that we successfully install docker by terminal command
```
 $ sudo docker version 
 $ sudo docker compose version
```


## Next: write a Dockerfile:

```
 $ vim Dockerfile
```
content in Dockerfile:
```
# Use an official Ubuntu image as the base OS
FROM ubuntu:20.04

#Set timezone and  environment as non-iteractive, avoid answering questions in R install
ENV DEBIAN_FRONTEND noninteractive
ENV TZ=America/Chicago
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Update the package list and install necessary packages
RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    wget 

RUN apt-get install -y \
    perl \ 
    unzip 
    
RUN apt-get install -y \
    zlib1g-dev \
    libbz2-dev \
    liblzma-dev \
    libcurl4-openssl-dev \
    libssl-dev \
    make \
    libncurses5-dev

# Install BWA
RUN mkdir /usr/bwa && \
    cd /usr/bwa && \
    wget https://github.com/lh3/bwa/releases/download/v0.7.17/bwa-0.7.17.tar.bz2 && \
    tar xvjf bwa-0.7.17.tar.bz2 && \
    cd bwa-0.7.17 && \
    make && \
    cp bwa /usr/local/bin

# install samtools 1.11
RUN wget https://github.com/samtools/samtools/releases/download/1.11/samtools-1.11.tar.bz2
RUN tar -xjf samtools-1.11.tar.bz2 -C /usr/local/bin/
RUN cd /usr/local/bin/samtools-1.11/ && ./configure
RUN cd /usr/local/bin/samtools-1.11/ && make
RUN cd /usr/local/bin/samtools-1.11/ && make install
ENV PATH="/usr/local/bin/samtools-1.11:${PATH}"
# Ensure the /usr/local/bin directory is in the PATH
ENV PATH="/usr/local/bin:${PATH}"

# Install Java 8
RUN apt-get update && \
    apt-get install -y openjdk-8-jdk

# Install R
RUN apt-get install --no-install-recommends -y r-base

# Download and extract Mauve
RUN wget https://darlinglab.org/mauve/snapshots/2015/2015-02-13/linux-x64/mauve_linux_snapshot_2015-02-13.tar.gz -O /tmp/mauve.tar.gz \
    && tar xzf /tmp/mauve.tar.gz -C /usr/local/ \
    && rm /tmp/mauve.tar.gz

# Edit Mauve configuration to point to Java installation
RUN sed -i 's/JAVA_CMD=java/JAVA_CMD=\/usr\/bin\/java/g' /usr/local/mauve_snapshot_2015-02-13/Mauve

# Make Mauve executable and add it to the PATH
RUN chmod +x /usr/local/mauve_snapshot_2015-02-13/Mauve \
    && ln -s /usr/local/mauve_snapshot_2015-02-13/Mauve /usr/local/bin/

# Install popoolation
RUN wget https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/popoolation/popoolation_1.2.2.zip
# Extract popoolation
RUN unzip popoolation_1.2.2.zip
RUN rm popoolation_1.2.2.zip

# Use RUN to install Python packages (numpy and pandas) via pip, Python's package manager
RUN pip3 install numpy pandas
```

## Then build the image by Dockerfile 

```
# in general, build image with the following command(give your own username, imagename, tag) 
$ docker build -t `username`/`imagename`:`tag` .
ex: 
docker build -t marissazhang/python_perl_saml_popoo:1.0 .
```
Look through all the error messages given by docker, and debug. Use the command to check whether the image is installed successfully
$ docker images

And run inside the Docker container to check the packages.

```
docker run -it --rm=true marissazhang/python_perl_saml_popoo:1.0 .
```

## Push the image to the DockerHub
login in into your account
```
$ docker login
```
push your tested image to the hub
```
$ docker push username/imagename:tag
```

