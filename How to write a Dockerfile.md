#Docker tutorial

###Launch a docker container:
```
docker run hello-world
```
An image is a filesystem and parameters to use at runtime. It doesn’t have state and never changes. A container is a running instance of an image. When you ran the command, Docker Engine:

checked to see if you had the hello-world software image
downloaded the image from the Docker Hub (more about the hub later)
loaded the image into the container and “ran” it.

**hello-world** is image name
###How to build my docker images ?

create folder custom_docker
```
mkdir custom_docker
```
create file Dockerfile inside custom_docker:
```
nano Dockerfile
```
in Dockerfile, put your command to  run new container:

example
- First is command 
```
FROM ubuntu
```
this command tells Docker which image your image is based on
Next, put your custom command to image to this image. Example:
```
RUN apt-get -y update && apt-get -y install git
RUN git clone https://github.com/cloudcomputinghust/CloudTestbed
```
Finnaly, build your images:
```
docker build -t #New_image_name #Path_to_Dockerfile
```