#Docker tutorial

###Launch a docker container:
```
docker run hello-world
```
view container status
```
docker inspect #container_name_or_container_id
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
###Launch a client command line inside a docker container:
We will start a container from ubuntu image and run it in command line mod by:
```
$ docker run --name #conainer_name -t -i ubuntu /bin/bash
```
with:
**docker run** runs a container.
**ubuntu** is the image you would like to run.
**-t** flag assigns a pseudo-tty or terminal inside the new container.
**-i** flag allows you to make an interactive connection by grabbing the standard in (STDIN) of the container.
**/bin/bash** launches a Bash shell inside our container.

after success, you will be switch to container command line 
```
root@d8c90c747c87:/# 
```
to quit container comand line (and containner will be stopped), type **exit**
to quit container commandline without stop container, using **ctrl+p then ctrl+q** according [stackoverflow](http://stackoverflow.com/questions/25267372/correct-way-to-detach-from-a-container-without-stopping-it)

use **docker** psto see running containers

use **docker ps -a** to see all containers

use **docker ps -l** to see lastest container

use **docker stop #container_name** to stop #container_name container

to resume work with a running container, use **docker attach #container_name**

to turn on a stopped container, use **docker start #cotainer_name**
###Launch a docker container with network port and command line
use:
```
docker run -p#local_port:docker_port -t -i #images_name
```
example:
```
docker run -p 5062:5000 -t -i cloud_test_bed
```
###working with images
to show image list, use **docker images**
to get an image from repository, use **docker pull #image_name**

####to create  images from container
first, we need to attach to a running container or start new running container.

second, we can modify any thing in running container ( install new package, clone project from github, etc...) to make container is differrent with base parent image

after modify, we can create new images from modified container by using **docker commit**
```
docker commit -m "#commit_message" -a "#author_name" #container_id_or_container_name
```
to remove docker image, user **docker rmi #image_id_or_image_name**

### to view docker run setting, use docker run --help
###docker networking

to view network list, use **docker network ls**
The network named bridge is default network selected if when start a container, that container need network

whenever you run a container, it will be connect to a docker network until this container stop.
when a container stop, it disconnect with docker network.
when a container running, it has a ip and connect to a docker network

to see container ip, use **docker inspect #container_id_or_container_name**

to see network status, use **docker network inspect #network_name_or_id**
to create a new network, use
```
docker network create -d bridge #network_name 
```
with -d specified network driver (default is **bridge**)

you can specified the network that new container will be connected to by add option **--network #network_name** when prepare command to start a container, example
```
docker run --network network_test --name c_n_y -t -i ubuntu /bin/bash
```
to connect a container to a network, use
```
docker network connect #network_name_or_id #container_name_or_id
```
**important!** a container can connect to many networks.

in a container, you can connect (ping, http, etc...) to another container which has ip in the same network with current container.

#Docker storage

###container data storage fundamental
First, we have a question, what is location which store data when we write data to a container
the answer is here [docker container and layer](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/#/container-and-layers):
 
 this website said:
 
 **Container and layers**
 
The major difference between a container and an image is the top writable layer. All writes to the container that add new or modify existing data are stored in this writable layer. When the container is deleted the writable layer is also deleted. The underlying image remains unchanged.

Because each container has its own thin writable container layer, and all changes are stored in this container layer, this means that multiple containers can share access to the same underlying image and yet have their own data state. The diagram below shows multiple containers sharing the same Ubuntu 15.04 image.

![enter image description here](https://docs.docker.com/engine/userguide/storagedriver/images/sharing-layers.jpg)

Important infomation source 
[docker images and container](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/#/container-and-layers)
[moving data between container and host](https://docs.oracle.com/cd/E37670_01/E75728/html/section_x54_32w_gp.html)

**Data volumes and the storage driver**
When a container is deleted, any data written to the container that is not stored in a data volume is deleted along with the container.


###data volume in a container
A data volume is a directory or file in the Docker host’s filesystem that is mounted directly into a container. Data volumes are not controlled by the storage driver. Reads and writes to data volumes bypass the storage driver and operate at native host speeds. You can mount any number of data volumes into a container. Multiple containers can also share one or more data volumes.

The diagram below shows a single Docker host running two containers. Each container exists inside of its own address space within the Docker host’s local storage area (/var/lib/docker/...). There is also a single shared data volume located at /data on the Docker host. This is mounted directly into both containers.
![Storage volume](https://docs.docker.com/engine/userguide/storagedriver/images/shared-volume.jpg)

Data volumes reside outside of the local storage area on the Docker host, further reinforcing their independence from the storage driver’s control. When a container is deleted, any data stored in data volumes persists on the Docker host.


**Important note:**
Vietnamese

Volume trong container là nơi data storage cho các container. Khi mount một volume vào container, thì container có thể truy cập được mọi dữ liệu có trong host ở địa chỉ mount point đó. Ví dụ, ta tạo một container mới bằng lệnh sau: **docker run -v /home/cong:/opt/cong --name volume_test -t -i ubuntu /bin/bash**. Lúc này, ta có thể thấy thư mục **/home/cong** được mount vào container ở vị trí **/opt/cong**. Gỉa sử trước khi mount, thư mục **/home/cong** đã có sẵn file **test.py** rồi, thì sau khi container được tạo, ở bên trong container ta đi tới thư mục **/opt/cong**, sẽ thấy có file **test.py** bên trong thư mục này.

Nếu bây gio, từ bên trong container ta thêm 1 file **test2.py** vào bên trong thư mục /opt/cong, thì từ bên ngoài host ta sẽ nhìn thấy thư mục test2.py có ở trong thư mục /home/cong. Điều tương tự xảy ra khi ta thay đổi bên trong thư mục /home/cong ở bên ngoài host như thế nào, thì bên trong container sẽ nhìn thấy sự thay đổi tương ứng.

Một volume có thể được chia sẻ sử dụng chung bởi nhiều container, và bất cứ sự thay đổi tới volume từ container nào cũng sẽ được các container khác chia sẻ chung volume đó nhìn thấy . Tương tự host cũng sẽ nhìn thấy.

nếu mountpoint của volume đã có sẵn ở trong image (ở trong ví dụ trên là **/opt/cong**) thì khi container được chạy lên, dữ liệu ở thư mục này từ images sẽ không được sử dụng mà bị ẩn đi, thay vào đó sẽ là dữ liệu từ volume bên ngoài host sẽ có trong mount point. Khi container được tắt đi, dữ liệu từ images sẽ được khôi phục.

resume english

to add a volume to a container, use **-v #host_path:#mount_path**

can use a file as a data volume instead a folder, example:
**docker run --rm -it -v ~/.bash_history:/root/.bash_history ubuntu /bin/bash**

###Creating and mounting a data volume container

this method used for create a container and use it as a data volume:

first, create a container and set it as a data volume. Example: 
```
docker create -v /dbdata --name db22 -t -i ubuntu /bin/bash
```
this command will create a container and set it as a data volume 

second, use this data volume in a another container:
```
docker run --volumes-from db22 --name mtx3 -t -i ubuntu /bin/bash
```

when this container is create, it has a folder in location **/dbdata**. it is mounpoint of data volume **db22**. You also make changes in this mounpoint. and **db22** data volume can use in many containers

**important **
If you remove containers that mount volumes, including the initial dbstore container, or the subsequent containers db1 and db2, the volumes will not be deleted. To delete the volume from disk, you must explicitly call docker rm -v against the last container with a reference to the volume. This allows you to upgrade, or effectively migrate data volumes between containers.
 
To backup, restore and revmove a data volume container, use tip in this tutorial:
https://docs.docker.com/engine/tutorials/dockervolumes/#/backup-restore-or-migrate-data-volumes
