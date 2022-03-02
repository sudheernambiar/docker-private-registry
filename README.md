# Private docker registry
## Description

Docker private registry gives us a platfor to keep all our custom build & tested docker images safely and securely to avoid any unwanted interaction with external world. There should be connectivity between all those servers involved in building and deploying processes. By doing this, we will be having a little more space with regard to our build processes, security, time spent on etc. This also gives us more flexibility and less time with regards to a deployment. Users once verified a stable image can be pushed in to the local private registry.

## Pre-Requests
- Need to install docker and docker-compose

### Docker installation 

```sh
yum install docker -y after docker installation, please start and enable it
```
### docker-compose installation

```sh
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/bin/docker-compose
sudo chmod +x /usr/bin/docker-compose
docker-compose version   
```
> You should have your own SSL certificate files. If you need a free 90 day certificate you may go to https://www.sslforfree.com/ 
> The created certs where placed on the /certs folder as below and created a HTTP authentication using httpd-tools
> create a new directory in the name of registry and create/copy all files needed(certificate files)
```
#Install the httpd-tools for using the htpasswd command.
yum install httpd-tools -y
#auth~/ htpasswd -Bc registry.password username   ### This will create a authentication file under auth folder. Replace the username with your needed one.
```
```
~]# tree
.
├── auth
│   └── registry.password
├── certs
│   ├── server.crt                         <<<<<<<<<<<<<<<<<<<  This is certificate.crt + ca_bundle.crt file
│   └── server.key
└── docker-compose.yml

2 directories, 4 files
```
```sh
version: '3'
services:
  registry:

    image: registry:2
    ports:
    - "5000:5000"

    environment:

      - REGISTRY_AUTH=htpasswd
      - REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm
      - REGISTRY_AUTH_HTPASSWD_PATH=/auth/registry.password
      - REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY=/data
      - REGISTRY_HTTP_ADDR=0.0.0.0:5000
      - REGISTRY_HTTP_TLS_CERTIFICATE=/certs/server.crt
      - REGISTRY_HTTP_TLS_KEY=/certs/server.key

    volumes:
      - data:/data
      - ./certs:/certs
      - ./auth:/auth

    networks:
      - registry_net

  frontend:
    environment:
     - ENV_DOCKER_REGISTRY_HOST=registry
     - ENV_DOCKER_REGISTRY_PORT=5000
     - ENV_DOCKER_REGISTRY_USE_SSL=1
     - ENV_USE_SSL="yes"
    volumes:
     - ./certs/server.crt:/etc/apache2/server.crt:ro
     - ./certs/server.key:/etc/apache2/server.key:ro
    ports:
     -  443:443
    image: konradkleine/docker-registry-frontend:v2
    networks:
     - registry_net

volumes:
  data:
networks:
  registry_net:
  ```
### How to run the compose
```
> docker-compose config              #--> To check the syntax

> docker-compose up -d               #--> This will create all the requirement which we have added on the yml file

> docker-compose ps   

### After creation
```
# docker-compose ps
       Name                      Command               State                      Ports
-----------------------------------------------------------------------------------------------------------
registry_frontend_1   /bin/sh -c $START_SCRIPT         Up      0.0.0.0:443->443/tcp,:::443->443/tcp, 80/tcp
registry_registry_1   /entrypoint.sh /etc/docker ...   Up      0.0.0.0:5000->5000/tcp,:::5000->5000/tcp

```
> How to push the image to our private registry. This has to be done from the remote machine you want to push image from.
```
Using a sample image form the docker hub of alpine to push it in our private registry
$ docker pull alpine:latest
$ docker tag alpine:latest  registry.oncompute.com:5000/bases/alpineimage:latest
$ docker login registry.oncompute.com:5000

Username: sudheer
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

$docker push registry.oncompute.com:5000/bases/alpineimage:latest
The push refers to repository [registry.oncompute.com:5000/bases/alpineimage]
8d3ac3489996: Pushed
latest: digest: sha256:e7d88de73db3d3fd9b2d63aa7f447a10fd0220b7cbf39803c803f2af9ba256b3 size: 528
```
> How to pull the image from our private registry
```
$docker pull registry.oncompute.com:5000/bases/alpineimage:latest
latest: Pulling from bases/alpineimage
Digest: sha256:e7d88de73db3d3fd9b2d63aa7f447a10fd0220b7cbf39803c803f2af9ba256b3
Status: Image is up to date for registry.oncompute.com:5000/bases/alpineimage:latest
registry.oncompute.com:5000/bases/alpineimage:latest

$docker images
REPOSITORY                                      TAG       IMAGE ID       CREATED        SIZE
alpine                                          latest    c059bfaa849c   3 months ago   5.59MB
registry.oncompute.com:5000/bases/alpineimage   latest    c059bfaa849c   3 months ago   5.59MB

You can see the second image
```
## Conclusion
Our private registry is now live with frontend.
We can access it with https://<yourdomain name>/<reponame>/<imagename>:<tag>

## Sample snaps

<center><img alt="auth-trigger" src="firefox_CqRyTV1ZYs.png"> </img></center>
<center><img alt="auth-trigger" src="firefox_un8ykpix0o.png"> </img></center>
<center><img alt="auth-trigger" src="firefox_LUegi7SjaD.png"> </img></center>

## Notes:
* Sometimes there may be certificate loading issue. Use the ca_bundle and certificate file to merge as one with server.crt (certificate contents on top of bundle contents and save as server.crt)
#### ⚙️ Connect with Me

