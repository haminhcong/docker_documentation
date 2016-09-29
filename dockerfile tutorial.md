f#Dockerfile tutorial
nguồn:
 https://www.digitalocean.com/community/tutorials/docker-explained-using-dockerfiles-to-automate-building-of-images

##1 What is a Dockerfile ?

Dockerfile is a file contains script that includes commands  (instructions) and arguments listed successively to automatic perform actions on a bases images in order to create new one. 

Dockerfiles begin with defining an image FROM which the build process starts. Followed by various other methods, commands and arguments (or conditions), in return, provide a new image which is to be used for creating docker containers.

They can be used by providing a Dockerfile's content - in various ways - to the docker daemon to build an image (as explained in the "How To Use" section).


##2 Dockerfile syntax
Cú pháp trong docker:

Định nghĩa syntax: Trong lập trình, syntax là các cấu trúc, các quy tắc để viết ra các chỉ dẫn (các lệnh).
các cấu trúc, các quy tắc này được định nghĩa 1 cách rõ ràng, người lập trình sẽ sử dụng các cấu trúc và quy tắc này để viết ra các lệnh cho chương trình hoặc các file chạy script. Nếu các quy tắc và các cấu trúc này không được tuân theo chính xác, chương trình sẽ không chạy.

Docker có một hệ thống syntax rõ ràng, đơn gỉan và dễ hiểu.  Chúng được thiết kế để người dùng có thể hiểu ý nghĩa ngay khi đọc mã (code) , hay nói cách khác là tự bản thân từng câu lệnh trong đoạn mã giai thích được chức năng của chúng. Thêm vào dó, docker file cho phép sử dụng các chú thích (comments) để diễn giai ý nghĩa của các câu lệnh. 

Dockerfile bao gồm 2 thành phần chủ yếu: các chú thích và các câu lệnh. Ví dụ:
```
# Print "Hello docker!"
RUN echo "Hello docker!"
```
Các câu lệnh được sử dụng trong Dockerfile được liệt kê phía dưới đây như sau:
###2.1 FROM
```
FROM <image>
```
The FROM instruction sets the Base Image for subsequent instructions. As such, a valid Dockerfile must have FROM as its first instruction. The image can be any valid image – it is especially easy to start by pulling an image from the Public Repositories.

https://docs.docker.com/engine/reference/builder/#/from
###2.2 MAINTAINER
```
MAINTAINER <name>
```
The MAINTAINER instruction allows you to set the Author field of the generated images.
https://docs.docker.com/engine/reference/builder/#/maintainer
###2.3 RUN
RUN có 2 form:
```
RUN <command> (shell form, the command is run in a shell, which by default is /bin/sh -c on Linux or cmd /S /C on Windows)

RUN ["executable", "param1", "param2"] (exec form)
```
RUN thực hiện các câu lệnh được truyền vào bên trong nó ở layer trên nhất của images, sau đó gộp (commit) dữ liệu chạy từ câu lệnh này vào images. Images sau khi được gộp layer sẽ được sử dụng ở các bước tiếp theo trong Dockerfile.

chúng ta sẽ sử dụng exec form khi câu lệnh truyền vào là phức tạp và có thể phá vỡ syntax của shell

Nếu câu lệnh trong RUN qúa dài, sử dụng dấu sổ chéo ```\```  (backslash) để xuống dòng. Ví dụ:
```
RUN /bin/bash -c 'source $HOME/.bashrc ;\
echo $HOME'
```
tương đương với lệnh sau:
```
RUN /bin/bash -c 'source $HOME/.bashrc ; echo $HOME'
```
###2.4 CMD
Lệnh CMD tương tự như lệnh RUN, được dùng để thực thi một câu lệnh. Tuy nhiên, lệnh RUN được thực thi trong qúa trình tạo images mới, còn lệnh CMD được thực thi khi chúng ta tạo một container mới thừ images mới này. Do đó, khi trong qúa trình tạo container mới từ images, lệnh CMD sẽ được chạy như những lệnh khởi tạo container.

Để giải thích rõ hơn về lệnh CMD, chúng ta có thể lấy ví dụ sau:
Lệnh RUN sẽ thực hiện các câu lệnh có vai trò tác động tới các dữ liệu có trong images hiện tại. Ví dụ images cơ sở là ubuntu, chưa có package python, flask và redis. Chúng ta sẽ sử dụng lệnh RUN để lấy các package này:
```
RUN apt-get install python
RUN apt-get install flask
RUN apt-get install redis
```
sau khi các lệnh RUN này được thực hiện, các package này sẽ được tải về images mới. Do đó khi images mới được tạo ra, trong images mới sẽ chứa các package trên.

Lệnh CMD được coi là các câu lệnh khởi tạo khi chúng ta tạo ra một container mới từ một images. Ví dụ trong images có packages flask và 1 website, và ta muốn sau khi container được khởi tạo thì package trên được kích hoạt, chúng ta sẽ dùng lệnh CMD để kích hoạt package trên:
```
CMD python app.py
```
lúc này, trong qúa trình container được khởi tạo, câu lệnh **python app.py** được thực thi và chạy ngay sau khi container mới được tạo ra.

Tóm lại, lệnh **RUN** dùng để thay đổi các dữ liệu và các gói có trong images mới, lệnh **CMD** để thiết lập các lệnh sẽ được chạy ( như bật 1 service, một package) sau khi container mới được tạo ra từ  images

Syntax của **CMD**
```
CMD ["executable","param1","param2"] (exec form, this is the preferred form)
CMD ["param1","param2"] (as default parameters to ENTRYPOINT)
CMD command param1 param2 (shell form)
```
Lưu ý về **CMD**:

 - Important: **There can only be one CMD instruction in a Dockerfile. If you list more than one CMD then only the last CMD will take effect.**

--> Nên chỉ có một lệnh CMD duy nhất trong 1 Dockerfile

 - The main purpose of a CMD is to provide defaults for an executing container. These defaults can include an executable, or they can omit the executable, in which case you must specify an ENTRYPOINT instruction as well.
 - If you would like your container to run the same executable every time, then you should consider using ENTRYPOINT in combination with CMD. See ENTRYPOINT.
 - don’t confuse RUN with CMD. RUN actually runs a command and commits the result; CMD does not execute anything at build time, but specifies the intended command for the image.
###2.5 LABEL
```
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```
The LABEL instruction adds metadata to an image. A LABEL is a key-value pair. To include spaces within a LABEL value, use quotes and backslashes as you would in command-line parsing. A few usage examples:

```
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \
that label-values can span multiple lines."
```
An image can have more than one label. To specify multiple labels, Docker recommends combining labels into a single LABEL instruction where possible. Each LABEL instruction produces a new layer which can result in an inefficient image if you use many labels. This example results in a single image layer.
https://docs.docker.com/engine/reference/builder/#/label
###2.6 EXPOSE
```
EXPOSE <port> [<port>...]
```
The EXPOSE instruction informs Docker that the container listens on the specified network ports at runtime. EXPOSE does not make the ports of the container accessible to the host. To do that, you must use either the -p flag to publish a range of ports or the -P flag to publish all of the exposed ports. You can expose one port number and publish it externally under another number.

Để một container có thể được truy cập từ bên ngoài container, chúng ta có các lựa chọn sau đây:
- Sử dụng EXPOSE để mở port, và không sử dụng flag `-p` trong câu lệnh `docker run`
- Sử dụng cả EXPOSE để mở port và sử dụng flag -p trong câu lệnh `docker run`

nếu như không sử dụng EXPOSE trong dockerfile cũng như không sử dụng flag -p  khi build một container mới, container mới được tạo ra sẽ không thể truy cập được từ bên ngoài.
Nếu sử dụng EXPOSE mà không sử dụng flag `-p` thì chỉ có CÁC CONTAINER kết nối với docker là có thể truy cập vào serivce của container ta mới tọa.
Nếu sửu dụng cả `-p` và cả EXPOSE, cả các container khác và từ bên ngoài host đều truy cập vào được
EXPOSE cho phép mở các port xác định trên container lúc nó được tạo ra. Tuy nhiên, chỉ các container khác trong docker mới có thể truy cập vào được các port này, còn từ bên ngoài host không truy cập vào được. Để từ host có thể truy cập được vào port, chúng ta cần cung cấp thêm tham số -p hoặc -P cho container lúc tạo ra nó.

PS: If you do -p, but do not EXPOSE, Docker does an implicit EXPOSE. This is because if a port is open to the public, it is automatically also open to other Docker containers. Hence -p includes EXPOSE. That's why I didn't list it above as a fourth case.

Trong trường hợp chúng ta run container từ một images không có lệnh EXPOSE, chúng ta vẫn có thể truy cập được từ bên ngoài host và từ các container khác vào host này nếu trong lệnh docker run xác định flag `-p`. Điều này xảy ra  là vì  khi chúng ta mở một port ở trên container là public bằng flag `-p`, thì chúng ta cũng mở port này cho các container khác. Do đó khi sử dụng flag `-p` nhưng không sử dụng EXPOSE, thì Docker sẽ thực hiện ngầm lệnh EXPOSE cho port được truyền vào flag `-p`
http://stackoverflow.com/questions/22111060/difference-between-expose-and-publish-in-docker
http://www.hoclaptrinh.org/hoi-dap/Khac-nhau-giua-Docker-expose-va-Docker-publish
###2.7 ENV
```
ENV <key> <value>
ENV <key>=<value> ...
```
The ENV instruction sets the environment variable <key> to the value <value>. This value will be in the environment of all “descendant” Dockerfile commands and can be replaced inline in many as well.

The ENV instruction has two forms. The first form, ENV <key> <value>, will set a single variable to a value. The entire string after the first space will be treated as the <value> - including characters such as spaces and quotes.

The second form, ENV <key>=<value> ..., allows for multiple variables to be set at one time. Notice that the second form uses the equals sign (=) in the syntax, while the first form does not. Like command line parsing, quotes and backslashes can be used to include spaces within values.

Example:
```
ENV myName="John Doe" myDog=Rex\ The\ Dog \
    myCat=fluffy
```
https://docs.docker.com/engine/reference/builder/#/env



###2.8 ADD
Lệnh ADD sử dụng để copy một folder từ máy chủ (host) vào bên trong images mới, với 2 tham số đầu vào là đường dẫn của thư mục cần copy ở host (**source foder path**) và đường dẫn tới thư mục này sau khi thư mục này được copy vào images mới (**destination folder path**). Ngoài ra, có thể truyền vào source folder path một URL , khi đó nội dung chứa ở URL sẽ được tải về và copy vào images mới.
```
ADD <src>... <dest>
ADD ["<src>",... "<dest>"] (this form is required for paths containing whitespace)
```

https://docs.docker.com/engine/reference/builder/#/add
###2.9 COPY
The COPY instruction copies new files or directories from <src> and adds them to the filesystem of the container at the path <dest>.
```
COPY <src>... <dest>
COPY ["<src>",... "<dest>"] (this form is required for paths containing whitespace)
```
Sự khác nhau giữa ADD và COPY:
```
ADD allows <src> to be an URL
If the <src> parameter of ADD is an archive in a recognised compression format, it will be unpacked
```
http://stackoverflow.com/questions/24958140/docker-copy-vs-add
https://docs.docker.com/engine/reference/builder/#/copy
###2.10 ENTRYPOINT
Định nghĩa từ wikipedia:
https://en.wikipedia.org/wiki/Entry_point
```
In computer programming, an entry point is where control is transferred from the operating system to a computer program, at which place the processor enters a program or a code fragment and execution begin
```
entrypoint được hiểu là nơi mà control điều khiển (ví dụ: dấu nhắc lệnh) chuyển từ hệ điều hành sang một chương trình đang thực thi.

Vai trò của câu lệnh ENTRYPOINT trong docker file:
```
An ENTRYPOINT allows you to configure a container that will run as an executable.
```
- RUN executes command(s) in a new layer and creates a new image. E.g., it is often used for installing software packages.
- CMD sets default command and/or parameters, which can be overwritten from command line when docker container runs.
- ENTRYPOINT configures a container that will run as an executable.
http://goinbigdata.com/docker-run-vs-cmd-vs-entrypoint/

Ví dụ:
```
FROM ubuntu
ENTRYPOINT ["top", "-b"]
CMD ["-c"]
```
sau khi câu lệnh này được chạy, màn hình của chương trình **top** hiện lên:
```
$ docker run -it --rm --name test  top -H
top - 08:25:00 up  7:27,  0 users,  load average: 0.00, 0.01, 0.05
Threads:   1 total,   1 running,   0 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.1 us,  0.1 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem:   2056668 total,  1616832 used,   439836 free,    99352 buffers
KiB Swap:  1441840 total,        0 used,  1441840 free.  1324440 cached Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
    1 root      20   0   19744   2336   2080 R  0.0  0.1   0:00.04 top
```

Ta cần lưu ý rằng:
```
Only the last ENTRYPOINT instruction in the Dockerfile will have an effect.
You can override the ENTRYPOINT instruction using the docker run --entrypoint flag.
```
Syntax của ENTRYPOINT:

```
ENTRYPOINT ["executable", "param1", "param2"]
ENTRYPOINT command param1 param2
```
Thông tin thêm về ENTRYPOINT:
https://docs.docker.com/engine/reference/builder/#/entrypoint
https://docs.docker.com/engine/reference/builder/#/understand-how-cmd-and-entrypoint-interact

###2.11 VOLUME
```
VOLUME ["<path>"]
```
The VOLUME instruction creates a mount point with the specified name and marks it as holding externally mounted volumes from native host or other containers
http://stackoverflow.com/questions/25311613/docker-mounting-volumes-on-host

Về cơ bản, câu lệnh bên trên tương đương với việc thêm flag ```-v <path>``` vào câu lệnh **docker run**
VOLUME ["<path>"] cho phép khởi tạo 1 volume mới khi chạy 1 container được build từ images tạo bởi dockerfile này. Đối với container, volume này có đường dẫn là ```<path>```, còn với host, volume này có đường dẫn là ```"Source": "/var/lib/docker/volumes/<volumes_id>```

Mục đích của câu lệnh này là tạo ra 1 volume trong container với đường dẫn là **path**  ngay khi container được tạo ra từ images mà chúng ta đang viết, sau  đó các container khác có thể sử dụng chung volume với container này bằng cách thêm flag:
```
--volumes-from <tên container chứa volume đầu tiên>
```
khi muốn 1 container mới tạo ra sử dụng chung volume với container đầu tiên.

###2.12 USER
```
USER #user_name
```
thiết lập tên (hoặc UID) của user thực hiện các lệnh CMD , RUN, ...
và user này cũng sẽ được sử dụng trong container được tạo ra từ images này

Khi chạy lệnh docker run  với flag``` --user="<user_name>" ```thì user_name phải là 1 trong các USER được khai báo trong dockerfile với câu lệnh ```USER #user_name ``` chứ không phải user của host.
https://github.com/docker/docker/issues/12514
###2.13 WORKDIR
```
WORKDIR /path/to/workdir
```
Thiết lập thư mục mà các lệnh CMD, RUN, ENTRYPOINT sẽ thực hiện các câu lệnh.

The WORKDIR instruction sets the working directory for any RUN, CMD, ENTRYPOINT, COPY and ADD instructions that follow it in the Dockerfile. If the WORKDIR doesn’t exist, it will be created even if it’s not used in any subsequent Dockerfile instruction.

It can be used multiple times in the one Dockerfile. If a relative path is provided, it will be relative to the path of the previous WORKDIR instruction. For example:
```
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```
The output of the final pwd command in this Dockerfile would be /a/b/c.

The WORKDIR instruction can resolve environment variables previously set using ENV. You can only use environment variables explicitly set in the Dockerfile. For example:

```
ENV DIRPATH /path
WORKDIR $DIRPATH/$DIRNAME
RUN pwd
```
The output of the final pwd command in this Dockerfile would be /path/$DIRNAME

https://docs.docker.com/engine/reference/builder/#/workdir
###2.14 ARG

    ARG <name>[=<default value>]
lệnh ARG cho phép định nghĩa các biến mà người dùng có thể gán gía trị tại thời điểm  build image với câu lệnh **docker build** bằng cách truyền gía trị vào flag `--build-arg <varname>=<value>`
If a user specifies a build argument that was not defined in the Dockerfile, the build outputs an error.
--> Nếu người dùng truyền một tham số (argument) chưa được định nghĩa trong Dockerfile vào `--build-arg <varname>=<value>`, thì chương trình build sẽ báo lỗi.
Ví dụ, với Dockerfile có nội dung như sau:
```
FROM ubuntu
ARG user1
RUN echo $user1
# Will output something like ===> 907ad6c2736f
#RUN apt-get -y update && apt-get -y install git
#RUN git clone https://github.com/cloudcomputinghust/CloudTestbed

```
Khi chạy lệnh `docker build --build-arg df=22 .`, trình build báo lỗi do trong docker file không chứa câu lệnh định  nghĩa biến **df** (tức là nếu muốn trình dịch không báo lỗi thì Dockerfile phải tồn tại câu lệnh khai báo biến **df** `ARG df` ):

```
devstack@devstack-pc:~$ docker build --build-arg df=22 .
Sending build context to Docker daemon 158.3 MB
Step 1 : FROM ubuntu
 ---> 45bc58500fa3
Step 2 : ARG user1
 ---> Running in 27010cabde5d
 ---> 7b59a38b2a17
Removing intermediate container 27010cabde5d
Step 3 : RUN echo $user1
 ---> Running in bab11e86b0ed

 ---> 00fd684909b6
Removing intermediate container bab11e86b0ed
One or more build-args [df] were not consumed, failing build.
```
Nếu chúng ta truyền vào câu lệnh `docker build` gía trị một biến đã được định nghĩa trong Dockerfile, trình build sẽ không báo lỗi. Ví dụ:
```
devstack@devstack-pc:~$ docker build --build-arg user1=22 .
Sending build context to Docker daemon 158.3 MB
Step 1 : FROM ubuntu
 ---> 45bc58500fa3
Step 2 : ARG user1
 ---> Using cache
 ---> 7b59a38b2a17
Step 3 : RUN echo $user1
 ---> Running in d33d131c5298
22
 ---> 666cd0afeeeb
Removing intermediate container d33d131c5298
Successfully built 666cd0afeeeb
```
Nếu khi build chúng ta không truyền gía trị cho các biến được định nghĩa trong Dockerfile, trình dịch sẽ vẫn chạy, lúc này các biến này sẽ có gía trị là gía trị mặc định.)
Trong ví dụ dưới đây, **ARG user1** sẽ nhận gía trị mặc định là rỗng:
```
devstack@devstack-pc:~$ docker build .
Sending build context to Docker daemon 158.3 MB
Step 1 : FROM ubuntu
 ---> 45bc58500fa3
Step 2 : ARG user1
 ---> Using cache
 ---> 38cc2c1bf9fa
Step 3 : RUN echo $user1
 ---> Running in 43d3d38484d9

 ---> 96de36e6e2cc
Removing intermediate container 43d3d38484d9
Successfully built 96de36e6e2cc
```

The Dockerfile author can define a single variable by specifying ARG once or many variables by specifying ARG more than once. For example, a valid Dockerfile:
```
FROM busybox
ARG user1
ARG buildno
...
```
thông tin thêm về ARG trong Dockerfile:
https://docs.docker.com/engine/reference/builder/#/arg

###2.15 ONBUILD
```
ONBUILD [INSTRUCTION]
```
ONBUILD is a new instruction for the Dockerfile. It is for use when creating a base image and you want to defer instructions to child images
The ONBUILD instruction adds to the image a trigger instruction to be executed at a later time, when the image is used as the base for another build. The trigger will be executed in the context of the downstream build, as if it had been inserted immediately after the FROM instruction in the downstream Dockerfile.

Any build instruction can be registered as a trigger.

This is useful if you are building an image which will be used as a base to build other images, for example an application build environment or a daemon which may be customized with user-specific configuration.

--> Giai thích:
Onbuild là công cụ cho phép chúng ta thiết lập các lệnh sẽ chạy khi build một dockerfile kế thừa dockerfile hiện tại.
Công cụ này sẽ rất hữu ích khi chúng ta xây dựng một images, cái mà sẽ được sử dụng làm images cơ sở để xây dựng các images khác. Ví dụ, images base sẽ là images chứa base là môi trường chạy ứng dụng, trong khi đó images kế thừa images này sẽ là images chứa ứng dụng.

Một ví dụ cụ thể.
http://container42.com/2014/02/06/docker-quicktip-3-onbuild/

Ví dụ 1:
Ở dockerfile của images được sử dụng làm base, ta nạp vào:
```
# Dockerfile của images cơ sở
FROM busybox
ONBUILD RUN echo "You won't see me until later"
```
khi tạo images  cơ sở từ dockerfile này, kết qủa hiện ra:
```
docker build -t me/no_echo_here .

Uploading context  2.56 kB
Uploading context
Step 0 : FROM busybox
Pulling repository busybox
769b9341d937: Download complete
511136ea3c5a: Download complete
bf747efa0e2f: Download complete
48e5f45168b9: Download complete
 ---&gt; 769b9341d937
Step 1 : ONBUILD RUN echo "You won't see me until later"
 ---&gt; Running in 6bf1e8f65f00
 ---&gt; f864c417cc99
Successfully built f864c417cc9
```

Chúng ta có thể thấy, lệnh `RUN echo "You won't see me until later"` không được thực thi khi build images cơ sở. Vậy lệnh này khi nào được thực hiện ?
vấn đề ở đây là, câu lệnh `ONBUILD RUN echo "You won't see me until later"` không thực hiện trong images cơ sở, mà công việc nó thực hiện là chỉ dẫn các images kế thừa images cơ sở thực hiện  lệnh `RUN echo "You won't see me until later` ngay sau khi câu lệnh FROM trong dockerfile của images kế thừa được thực hiện.
Ví dụ, khi ta viết một dockerfile kế thừa images cơ sở:
```
# Dockerfile kế thừa images cơ sở
FROM me/no_echo_here
```
sau đó build images từ dockerfile kế thừa này:
```
docker build -t me/echo_here .
Uploading context  2.56 kB
Uploading context
Step 0 : FROM cpuguy83/no_echo_here

# Executing 1 build triggers
Step onbuild-0 : RUN echo "You won't see me until later"
 ---&gt; Running in ebfede7e39c8
You won't see me until later
 ---&gt; ca6f025712d4
 ---&gt; ca6f025712d4
Successfully built ca6f025712d4
```

chúng ta có thể thấy, khi chúng ta chạy lệnh `docker build` để build images từ dockerfile kế thừa images cơ sở, lệnh ` RUN echo "You won't see me until later"` được thực thi.

Ví dụ 2:
Dockerfile của images cơ sở:
```
# Dockerfile
FROM ubuntu:12.04

RUN apt-get update -qq &amp;&amp; apt-get install -y ca-certificates sudo curl git-core
RUN rm /bin/sh &amp;&amp; ln -s /bin/bash /bin/sh

RUN locale-gen  en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
ENV LC_ALL en_US.UTF-8

RUN curl -L https://get.rvm.io | bash -s stable
ENV PATH /usr/local/rvm/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
RUN /bin/bash -l -c rvm requirements
RUN source /usr/local/rvm/scripts/rvm &amp;&amp; rvm install ruby
RUN rvm all do gem install bundler

ONBUILD ADD . /opt/rails_demo
ONBUILD WORKDIR /opt/rails_demo
ONBUILD RUN rvm all do bundle install
ONBUILD CMD rvm all do bundle exec rails server
```
Dockerfile này thực hiện các thiết lập môi trường trong images cơ sở, sau đó nó thiết lập các lệnh `ONBUILD`  để hướng dẫn các Dockerfile kế thừa images cơ sở này thực hiện các lệnh sau khi build các images dẫn xuất:
```
ONBUILD ADD . /opt/rails_demo
ONBUILD WORKDIR /opt/rails_demo
ONBUILD RUN rvm all do bundle install
ONBUILD CMD rvm all do bundle exec rails server
```
ở đây, khi các images dẫn xuất bắt đầu được build, các lệnh sau được thực hiện:
ONBUILD ADD .  /opt/rails_demo Tells any child image to add everything in the current directory to /opt/railsdemo. Remember, this only gets run from a child image, that is when another image uses this one as a base (or FROM). And it just so happens if you look in the repo I have a skeleton rails app in railsdemo that has it's own Dockerfile in it, we'll take a look at this later.
-->lệnh ONBUILD đầu tiên hướng dẫn **tiến trình build** images dẫn xuất thêm mọi data từ thư mục hiện tại vào images dẫn xuất với đường dẫn đích là ` /opt/rails_demo`. 

ONBUILD WORKDIR /opt/rails_demo Tells any child image to set the working directory to /opt/rails_demo, which is where we told ADD to put any project files
--> lệnh ONBUILD thứ hai chỉ dẫn tiến trình build images dẫn xuất thiế lập thư mục chạy (working directory) là ` /opt/rails_demo`
ONBUILD RUN rvm all do bundle install Tells any child image to have bundler install all gem dependencies, because we are assuming a Rails app here.

--> lệnh ONBUILD thứ ba hướng dẫn tiến trình build images dẫn xuất cài đặt mọi packages cần thiết vào images dẫn xuất  để cài đặt môi trường cho Rails

ONBUILD CMD rvm all do bundle exec rails server Tells any child image to set the CMD to start the rails server
--> lệnh ONBUILD thứ tư hướng dẫn tiến trình build images dẫn xuất thiết lập lệnh CMD cho images dẫn xuất, lúc này mọi container được khởi tạo từ images dẫn xuất sẽ khởi động rails server ngay sau khi container đó được tạo ra.

Do ở Dockerfile cho images cơ sở đã thiết lập mọi lệnh cần thiết cho qúa trình build images dẫn xuất, do đó, ở Dockerfile kế thừa images cơ sở (hay kế thừa Dockerfile cơ sở), chúng ta đơn giản là chỉ cần import images cơ sở, và chuẩn bị các tài nguyên cần thiết để build rail server trong current directory.
```
# Dockerfile cho images dẫn xuất từ images cơ sở
FROM cpuguy83/onbuild_demo
```
WAT?? This Dockerfile is a grand total of one line. It's only one line because we setup everything in the base image. The only pre-req is that the Dockerfile is built from within the Rails project tree. When we build this image, the ONBUILD commands from cpuguy83/onbuild_demo will be inserted just after the FROM instruction here.

Remember, this aggressive use of ONBUILD may not be optimal for your project and is for demo purposes... not to say it's not ok :)

kết qủa khi build images dẫn xuất bằng dockerfile trên 
```
cd rails_demo
docker build -t cpuguy83/rails_demo .

Step onbuild-0 : ADD . /opt/rails_demo
 ---&gt; 11c1369a8926
Step onbuild-1 : WORKDIR /opt/rails_demo
 ---&gt; Running in 82def1878360
 ---&gt; 39f8280cdca6
Step onbuild-2 : RUN rvm all do bundle install
 ---&gt; Running in 514d5fc643f1
# output supressed
Step onbuild-3 : CMD rvm all do bundle exec rails server
 ---&gt; Running in df4a2646e4d9
 ---&gt; b78c1813bd44
 ---&gt; b78c1813bd44
Successfully built b78c1813bd44
```

Kết qủa khi chạy một container bằng images dẫn xuất:
```
docker run -i -t cpuguy83/rails_demo

=&gt; Booting WEBrick
=&gt; Rails 3.2.14 application starting in development on http://0.0.0.0:3000
=&gt; Call with -d to detach
=&gt; Ctrl-C to shutdown server
[2014-02-06 11:53:20] INFO  WEBrick 1.3.1
[2014-02-06 11:53:20] INFO  ruby 2.1.0 (2013-12-25) [x86_64-linux]
[2014-02-06 11:53:20] INFO  WEBrick::HTTPServer#start: pid=193 port=3000
```
TLDR; ONBUILD... awesome. Use it to defer build instructions to images built from a base image. Use it to more easily build images from a common base but differ in some way, such as different git branches, or different projects entirely.

With great power comes great responsibility.

Nguồn:
http://stackoverflow.com/questions/34863114/dockerfile-onbuild-instruction
http://container42.com/2014/02/06/docker-quicktip-3-onbuild/

###2.16 STOPSIGNAL
https://docs.docker.com/engine/reference/builder/#/stopsignal
###2.17 HEALTHCHECK
https://docs.docker.com/engine/reference/builder/#/healthcheck
###2.18 SHELL
https://docs.docker.com/engine/reference/builder/#/shell

3 Lệnh trên chưa có điều kiện để tìm hiểu do thời gian có hạn.
