# A Demo app for Docker Startup
If you are new to docker and wanted to try out, but don't have any application in mind to set this up ... then you are at right place to get started. 
In this demo app, don't expect any good looking frontend or some mind blowing css, html ...it just a sufficient code to solve the purpose of running a Docker container... once you set this up and understand you are free to modify it as per your interest. 

# What you will get at end of this ?
You will get an Docker Installed on system with two container running one for PHP and another for Solr, and your PHP code is able to connect with Solr core.  

# System consideration 
 - Ubuntu 16.04.4 LTS

## Docker Installation
```bash
ubuntu@ip-xx-xx-xx:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
ubuntu@ip-xx-xx-xx:~$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
ubuntu@ip-xx-xx-xx:~$ sudo apt-get update
ubuntu@ip-xx-xx-xx:~$ apt-cache policy docker-ce
ubuntu@ip-xx-xx-xx:~$ sudo apt-get install - y docker-ce
ubuntu@ip-xx-xx-xx:~$ docker run hello-world
```

The last command `docker run hello-world` is to test the docker installation.

Now we will setup a Solr Container on this machine.

# Solr container setup
```bash
ubuntu@ip-xx-xx-xx:~$ docker run -it -d --name my-solr-1 -p 8983:8983 solr
```
The above command will check for `solr` image on your system first, if it does't find that it will download it from docker hub and run it for you. 
The default behavior of this container is to run solr on `127.0.0.1` interface i.e. localhost. If you want to change this behavior then create a set-host.sh file as mentioned below. 
```bash
#!/bin/bash
set -e
cp /opt/solr/bin/solr.in.sh /opt/solr/bin/solr.in.sh.orig
sed -e 's/SOLR_HEAP=".*"/SOLR_HEAP="1024m"/' </opt/solr/bin/solr.in.sh.orig >/opt/solr/bin/solr.in.sh
sed -e 's/SOLR_HOST=".*"/SOLR_HOST="0.0.0.0"/' </opt/solr/bin/solr.in.sh.orig >/opt/solr/bin/solr.in.sh
sed -e 's/SOLR_OPTS=".*"/SOLR_OPTS="$SOLR_OPTS -Djetty.host=0.0.0.0"/' </opt/solr/bin/solr.in.sh.orig >/opt/solr/bin/solr.in.sh

grep '^SOLR_HEAP=' /opt/solr/bin/solr.in.sh
grep '^SOLR_HOST=' /opt/solr/bin/solr.in.sh
grep '^SOLR_OPTS=' /opt/solr/bin/solr.in.sh
``` 
This file you need to provide to docker command while starting a Solr container. You will get an example of calling this file in upcoming section.

If you want to create a **core** of your interest, run below command.
```bash
ubuntu@ip-xx-xx-xx:~$ docker run --name my-solr-1 -d -P -v $PWD/set-host.sh -p 8980:8983 solr solr-precreate techproducts
```
In above command the argument list goes as below,
-d : detached
-v : volume
-p : Port <host:container>
-P : Publish all exposed ports to random ports
solr-precreate : Command to Solr container
techproducts : Name of the core

**Note:** When you create a Core, inside container the core got created at `/opt/solr/server/solr/mycores/<core-name>` location. 

The core get create but without any data. To add a sample data from *standard Solr examples* 
```bash
ubuntu@ip-xx-xx-xx:~$ docker exec -it --user=solr my-solr-1 bin/post -c techproducts example/exampledocs/manufacturers.xml
```

Instead of example data, if you want to add your own index data to this core, you can do that easily by executing below commands 

```bash
ubuntu@ip-xx-xx-xx:~$ docker run --name my-solr-1 -d -P -v $PWD/set-host.sh -p 8980:8983 solr solr-precreate techproducts
ubuntu@ip-xx-xx-xx:~$ docker cp data/ my-solr-1:/opt/solr/server/solr/mycores/techproducts/
ubuntu@ip-xx-xx-xx:~$ docker stop my-solr-1
ubuntu@ip-xx-xx-xx:~$ docker start my-solr-1
```
**Assumptions:**
Index data is available in `data` directory. Remember the “data” directory should have “*index*, *snapshot_metadata*, *tlog*” folders available.

# PHP container setup
The required details of setting up this container is captured inside `Dockerfile` make sure you have this git repo clone and available on your system.

## Build Dockerfile
```bash
ubuntu@ip-xx-xx-xx:~$ docker build -t jidnesh/php-app: v1.1 .
```
If all work well, you should get output for “check1.php”

