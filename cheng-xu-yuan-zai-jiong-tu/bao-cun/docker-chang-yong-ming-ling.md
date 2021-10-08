# Docker常用命令

一、帮助指令

docker version          \#显示docker 的版本信息

docker info             \#显示docker 的系统信息，包括镜像和容器的数量

docker \[命令\] --help    \#命令的 帮助信息

二、镜像命令

【docker images          \#查看镜像列表】

\#解释

REPOSITORY   TAG       IMAGE ID       CREATED       SIZE

REPOSITORY    \#镜像的仓库源

TAG           \#标签

IMAGE ID      \#镜像ID

CREATED       \#创建时间

SIZE          \#镜像大小

* --all 或 -a            \#看到所有的镜像

\[ByCore@localhost ~\]$ sudo docker images -a

REPOSITORY   TAG       IMAGE ID       CREATED       SIZE

&lt;none&gt;       &lt;none&gt;    6084105296a9   2 weeks ago   133MB

alpine       latest    28f6e2705743   5 weeks ago   5.61MB

* --filter 或 -f         \#过滤

**--filter=reference='\*:latest' \#匹配 所有 REPOSITORY / TAG 为 latest 的 镜像**

\[ByCore@localhost ~\]$ sudo docker images --filter=reference='\*:latest'

REPOSITORY   TAG       IMAGE ID       CREATED        SIZE

alpine       latest    302aba9ce190   21 hours ago   5.61MB

mysql        latest    26d0ac143221   7 days ago     546MB

nginx        latest    6084105296a9   2 weeks ago    133MB

sudo docker inspect --format '{{\(index .RootFS.Layers 0\)}}' 302aba9ce190

* --no-trunc              \#显示完整的镜像信息

\[ByCore@localhost ~\]$ sudo docker images --no-trunc

REPOSITORY   TAG       IMAGE ID                                                                  CREATED       SIZE

&lt;none&gt;       &lt;none&gt;    sha256:6084105296a952523c36eea261af38885f41e9d1d0001b4916fa426e45377ffe   2 weeks ago   133MB

alpine       latest    sha256:28f6e27057430ed2a40dbdd50d2736a3f0a295924016e294938110eeb8439818   5 weeks ago   5.61MB

\[ByCore@localhost ~\]$ sudo docker images

REPOSITORY   TAG       IMAGE ID       CREATED       SIZE

&lt;none&gt;       &lt;none&gt;    6084105296a9   2 weeks ago   133MB

alpine       latest    28f6e2705743   5 weeks ago   5.61MB

* --digests                \#显示镜像的摘要信息

\[ByCore@localhost ~\]$ sudo docker images --digests

REPOSITORY   TAG       DIGEST                                                                    IMAGE ID       CREATED       SIZE

&lt;none&gt;       &lt;none&gt;    &lt;none&gt;                                                                    6084105296a9   2 weeks ago   133MB

alpine       latest    sha256:a75afd8b57e7f34e4dad8d65e2c7ba2e1975c795ce1ee22fa34f8cf46f96a3be   28f6e2705743   5 weeks ago   5.61MB

* --quiet 或 -q          \#值显示id

\[ByCore@localhost ~\]$ sudo docker images -q

6084105296a9

28f6e2705743

【docker search mysql        \#搜索镜像】

\[ByCore@localhost ~\]$ sudo docker search mysql

NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED

mysql                             MySQL is a widely used, open-source relation…   10665     \[OK\]       

mariadb                           MariaDB Server is a high performing open sou…   3997      \[OK\]       

mysql/mysql-server                Optimized MySQL Server Docker images. Create…   779                  \[OK\]

percona                           Percona Server is a fork of the MySQL relati…   528       \[OK\]       

centos/mysql-57-centos7           MySQL 5.7 SQL database server                   87                   

mysql/mysql-cluster               Experimental MySQL Cluster Docker images. Cr…   80                   

centurylink/mysql                 Image containing mysql. Optimized to be link…   59                   \[OK\]

bitnami/mysql                     Bitnami MySQL Docker Image                      50                   \[OK\]

databack/mysql-backup             Back up mysql databases to... anywhere!         42                   

deitch/mysql-backup               REPLACED! Please use http://hub.docker.com/r…   41                   \[OK\]

prom/mysqld-exporter                                                              37                   \[OK\]

....

* --filter=STARS=3000        \#搜索 镜像STARS 大于3000 的镜像

\[ByCore@localhost ~\]$ sudo docker search mysql --filter=STARS=3000

NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED

mysql     MySQL is a widely used, open-source relation…   10665     \[OK\]       

mariadb   MariaDB Server is a high performing open sou…   3997      \[OK\]       

\[ByCore@localhost ~\]$

【docker pull 镜像名\[:tag\]        \#下载对应镜像】

\#不添加 TAG 下载最新版本

\[ByCore@localhost ~\]$ sudo docker pull mysql

Using default tag: latest

latest: Pulling from library/mysql

6f28985ad184: Already exists

e7cd18945cf6: Pull complete

ee91068b9313: Pull complete

b4efa1a4f93b: Pull complete

f220edfa5893: Pull complete

74a27d3460f8: Pull complete

2e11e23b7542: Pull complete

fbce32c99761: Pull complete

08545fb3966f: Pull complete

5b9c076841dc: Pull complete

ef8b369352ae: Pull complete

ebd210f0917d: Pull complete

Digest: sha256:5d1d733f32c28d47061e9d5c2b1fb49b4628c4432901632a70019ec950eda491

Status: Downloaded newer image for mysql:latest

docker.io/library/mysql:latest

\[ByCore@localhost ~\]$

\#列表显示容器的所有版本

\#添加 TAG 下载指定版本

\[ByCore@localhost ~\]$ sudo docker pull mysql:5.7

\[sudo\] password for ByCore:

Sorry, try again.

\[sudo\] password for ByCore:

5.7: Pulling from library/mysql

6f28985ad184: Already exists        =====&gt;

e7cd18945cf6: Already exists        =====&gt;

ee91068b9313: Already exists        =====&gt;

b4efa1a4f93b: Already exists        =====&gt;

f220edfa5893: Already exists        =====&gt;

74a27d3460f8: Already exists        =====&gt;

2e11e23b7542: Already exists        =====&gt; \#容器的分层架构，下载镜像时 会比对本地镜像的分层与Docker Hub 中的分层，如果存在折不下载

39ac93d44c47: Pull complete

dfd9db50d4ea: Pull complete

3bc1b061dd92: Pull complete

51da5253cefc: Pull complete

Digest: sha256:90284763b95c67f1bfb6e4de640ec3ceb82a45a12af321201cd51d62e722af1d

Status: Downloaded newer image for mysql:5.7

docker.io/library/mysql:5.7

\[ByCore@localhost ~\]$

【docker rmi \[镜像ID:镜像名\[:tag\] \]       \#删除对应镜像】

\#通过ID 删除镜像  \#通过镜像ID 删除 不能带TAG

\[ByCore@localhost ~\]$ sudo docker images

REPOSITORY   TAG       IMAGE ID       CREATED       SIZE

mysql        5.7       2fb283157d3c   7 days ago    449MB

mysql        latest    26d0ac143221   7 days ago    546MB

&lt;none&gt;       &lt;none&gt;    6084105296a9   2 weeks ago   133MB

alpine       latest    28f6e2705743   5 weeks ago   5.61MB

\[ByCore@localhost ~\]$ sudo docker rmi -f 6084105296a9    \#镜像有运行的容器时，不能够删除

Error response from daemon: conflict: unable to delete 6084105296a9 \(cannot be forced\) - image is being used by running container 1fa1073b3ea4

\[ByCore@localhost ~\]$ sudo docker rmi -f 2fb283157d3c

Untagged: mysql:5.7

Untagged: mysql@sha256:90284763b95c67f1bfb6e4de640ec3ceb82a45a12af321201cd51d62e722af1d

Deleted: sha256:2fb283157d3c46af56502f59004fddbf4f8a2531c5c1c8be6168233dd7a41939

Deleted: sha256:4a0e2fca8f43fdc427996d49f3ef1bc020d8da5183d986b81f7f302a391ae5ed

Deleted: sha256:e35ba21a7d23155e942328d1aa816016ae1663165e1431a34a74ca172730d650

Deleted: sha256:6d2efff120dd67002624d7999876db66afeaf2aead8b7ccee4453e0d21a99628

Deleted: sha256:ad80b667eed27f25e7066ea2ebe8e4449fd997d6011f8e24dc3e5c74a7a6784e

\[ByCore@localhost ~\]$

\#通过镜像名 删除镜像

\[ByCore@localhost ~\]$ sudo docker rmi mysql:latest

Untagged: mysql:latest

Untagged: mysql@sha256:5d1d733f32c28d47061e9d5c2b1fb49b4628c4432901632a70019ec950eda491

Deleted: sha256:26d0ac143221341c36402a139826e938d2ea6f2e458005a71699975c84e96ade

Deleted: sha256:16f5b1eb2e7319e8a0db5df7f1ee0903033400a42264fcfbcc2d946b12267895

Deleted: sha256:303119686434550f3672a755dcda8a0468d34472d77e8789ffef5dc5f73dc790

Deleted: sha256:88f159cadb30aacd4df26c9fb6e1fb71b3cc3f5ce05468659879216e7751bad7

Deleted: sha256:55b6e8ee7cbea49773b2a88c3941ebad16537df99b087e673ca4b0175ade1b70

Deleted: sha256:e27b1c89d3f9e194c1a3495c24a9546135ee3ab6625e94eaaedd09a41343e7d0

Deleted: sha256:c1e6768ecc0af349b2167ea29168a32da846ea4fd2ba08f37d4e45d9ee00080a

Deleted: sha256:1a0d8068bd29db71d09ff06b5356f73786f70bac70d157d2f4a1197a85aa682c

Deleted: sha256:4a52e2416c7889937a7dd82d485cc878f4d8607f20c0e77c4b0b060de457b00b

Deleted: sha256:cea5c46b3aee78a2d584f0374ab28a35995036de6b204086d34da265988e1970

Deleted: sha256:3e8ad860e72cc47bcdee6afa9c58904bbd1bc003acecaaf98393dd1c960026a6

Deleted: sha256:6f2660ea9ddbf232748e123ca4aa9f5f0ce40fef502ae2187456982b7c2efc33

\[ByCore@localhost ~\]$

\[ByCore@localhost ~\]$ sudo docker images        

REPOSITORY   TAG       IMAGE ID       CREATED       SIZE

nginx        latest    6084105296a9   2 weeks ago   133MB

alpine       latest    28f6e2705743   5 weeks ago   5.61MB

\[ByCore@localhost ~\]$ sudo docker ps

CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                NAMES

1fa1073b3ea4   nginx     "/docker-entrypoint.…"   55 minutes ago   Up 55 minutes   0.0.0.0:81-&gt;80/tcp   nginx-run

\[ByCore@localhost ~\]$ sudo docker rmi -f nginx:latest    \#删除 有容器运行的 镜像时，只时Untagged 对应的 REPOSITORY 和 TAG

Untagged: nginx:latest

\[ByCore@localhost ~\]$

\[ByCore@localhost ~\]$ sudo docker rmi nginx:latest   \#不加 -f 参数时 不允许删除

Error response from daemon: conflict: unable to remove repository reference "nginx:latest" \(must force\) - container 1fa1073b3ea4 is using its referenced image 6084105296a9

\[ByCore@localhost ~\]$

\#批量删除镜像

\[ByCore@localhost ~\]$ sudo docker rmi -f $\(docker images -qa\)    \#删除所有镜像 如果有其中某些镜像有容器运行，无法删除

\[ByCore@localhost ~\]$ sudo docker ps -a

CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS                        PORTS                NAMES

7a5830b10086   nginx     "/docker-entrypoint.…"   13 minutes ago   Exited \(137\) 12 minutes ago                        nginx-run2

23810f7ccd2a   nginx     "/docker-entrypoint.…"   19 minutes ago   Up 18 minutes                 0.0.0.0:81-&gt;80/tcp   nginx-run

\[ByCore@localhost ~\]$ sudo docker rm -f $\(sudo docker ps -qa --filter=status=exited\)  \#删除所有状态 为 exited 的容器

7a5830b10086

\#删除 tag 为 &lt;none&gt; 的镜像

\[ByCore@localhost ~\]$ sudo docker image prune

\[sudo\] password for ByCore:

WARNING! This will remove all dangling images.

Are you sure you want to continue? \[y/N\] y

Total reclaimed space: 0B

\[ByCore@localhost ~\]$

\#删除 无容器 使用的镜像

\[ByCore@localhost ~\]$ sudo docker images -a

REPOSITORY   TAG       IMAGE ID       CREATED       SIZE

mysql        5.7       2fb283157d3c   7 days ago    449MB

mysql        latest    26d0ac143221   7 days ago    546MB

nginx        latest    6084105296a9   2 weeks ago   133MB

alpine       latest    28f6e2705743   5 weeks ago   5.61MB

\[ByCore@localhost ~\]$ sudo docker image prune -a

WARNING! This will remove all images without at least one container associated to them.

Are you sure you want to continue? \[y/N\] y

Deleted Images:

untagged: mysql:5.7

untagged: mysql@sha256:90284763b95c67f1bfb6e4de640ec3ceb82a45a12af321201cd51d62e722af1d

deleted: sha256:2fb283157d3c46af56502f59004fddbf4f8a2531c5c1c8be6168233dd7a41939

deleted: sha256:4a0e2fca8f43fdc427996d49f3ef1bc020d8da5183d986b81f7f302a391ae5ed

deleted: sha256:e35ba21a7d23155e942328d1aa816016ae1663165e1431a34a74ca172730d650

deleted: sha256:6d2efff120dd67002624d7999876db66afeaf2aead8b7ccee4453e0d21a99628

deleted: sha256:ad80b667eed27f25e7066ea2ebe8e4449fd997d6011f8e24dc3e5c74a7a6784e

untagged: mysql:latest

untagged: mysql@sha256:5d1d733f32c28d47061e9d5c2b1fb49b4628c4432901632a70019ec950eda491

deleted: sha256:26d0ac143221341c36402a139826e938d2ea6f2e458005a71699975c84e96ade

deleted: sha256:16f5b1eb2e7319e8a0db5df7f1ee0903033400a42264fcfbcc2d946b12267895

deleted: sha256:303119686434550f3672a755dcda8a0468d34472d77e8789ffef5dc5f73dc790

deleted: sha256:88f159cadb30aacd4df26c9fb6e1fb71b3cc3f5ce05468659879216e7751bad7

deleted: sha256:55b6e8ee7cbea49773b2a88c3941ebad16537df99b087e673ca4b0175ade1b70

deleted: sha256:e27b1c89d3f9e194c1a3495c24a9546135ee3ab6625e94eaaedd09a41343e7d0

deleted: sha256:c1e6768ecc0af349b2167ea29168a32da846ea4fd2ba08f37d4e45d9ee00080a

deleted: sha256:1a0d8068bd29db71d09ff06b5356f73786f70bac70d157d2f4a1197a85aa682c

deleted: sha256:4a52e2416c7889937a7dd82d485cc878f4d8607f20c0e77c4b0b060de457b00b

deleted: sha256:cea5c46b3aee78a2d584f0374ab28a35995036de6b204086d34da265988e1970

deleted: sha256:3e8ad860e72cc47bcdee6afa9c58904bbd1bc003acecaaf98393dd1c960026a6

deleted: sha256:6f2660ea9ddbf232748e123ca4aa9f5f0ce40fef502ae2187456982b7c2efc33

untagged: alpine:latest

untagged: alpine@sha256:a75afd8b57e7f34e4dad8d65e2c7ba2e1975c795ce1ee22fa34f8cf46f96a3be

deleted: sha256:28f6e27057430ed2a40dbdd50d2736a3f0a295924016e294938110eeb8439818

deleted: sha256:cb381a32b2296e4eb5af3f84092a2e6685e88adbc54ee0768a1a1010ce6376c7

Total reclaimed space: 795.9MB

\[ByCore@localhost ~\]$ sudo docker images -a

REPOSITORY   TAG       IMAGE ID       CREATED       SIZE

nginx        latest    6084105296a9   2 weeks ago   133MB

三、容器命令

【docker run \[参数\] image】

\#参数说明

--name="name"        \#容器名称，给容器一个别名

-d                               \#后台方式运行

-it                               \#使用交互式运行，bind 交互式shell（进入交互式shell）

-p                               \#绑定（映射）宿主机端口到容器软口 ：-p  8080:8080

                  \#\[-p ip:主机端口:容器端口\] : 带 ip

                                 \#\[-p 主机端口:容器端口\]  


                  \#\[-p 容器端口\] : 不映射端口

                  \#\[容器端口\]

-P                               \#大P 随机指定端口

* 启动并进入容器

\[ByCore@localhost ~\]$ sudo docker run -it centos /bin/bash    \#不使用 --name 指定名称,docker 会随机生成一个 容器名称

\[sudo\] password for ByCore:

\[root@79342bff2ffb /\]\#

\[root@79342bff2ffb /\]\# exit    

exit

\[ByCore@localhost ~\]$ sudo docker ps

CONTAINER ID   IMAGE          COMMAND                  CREATED       STATUS       PORTS                NAMES

23810f7ccd2a   6084105296a9   "/docker-entrypoint.…"   2 hours ago   Up 2 hours   0.0.0.0:81-&gt;80/tcp   nginx-run

\[ByCore@localhost ~\]$ sudo docker ps -a

CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS                      PORTS                NAMES

79342bff2ffb   centos         "/bin/bash"              2 minutes ago   Exited \(0\) 14 seconds ago                        silly\_pike

23810f7ccd2a   6084105296a9   "/docker-entrypoint.…"   2 hours ago     Up 2 hours                  0.0.0.0:81-&gt;80/tcp   nginx-run

\[ByCore@localhost ~\]$

【docker ps】列表容器

\#参数说明

-a                      :显示所有的容器，包括未运行的。

-f                      :根据条件过滤显示的内容。

--format                :指定返回值的模板文件。 --format 的所有 key值 可以通过 模板 ：'{{json .}}' 来获取

-l                      :显示最近创建的容器。

-n                      :列出最近创建的n个容器。

--no-trunc              :不截断输出。

-q                      :静默模式，只显示容器编号。

-s                      :显示总的文件大小。

\#输出信息：

CONTAINER ID            : 容器 ID。

IMAGE                   : 使用的镜像。

COMMAND                 : 启动容器时运行的命令。

CREATED                 : 容器的创建时间。

STATUS                  : 容器状态。

\#状态类型：

created                 :（已创建）

restarting              :（重启中）

running                 :（运行中）

removing                :（迁移中）

paused                  :（暂停）

exited                  :（停止）

dead                    :（死亡）

\#过滤条件：

id                      : container's id \[容器ID\]

label                   : label=&lt;key&gt; \| label=&lt;key&gt;=&lt;value&gt;  \[1：  label=color   2:   label=color=blue\]

exited                  :

status                  : created\|restarting\|running\|removing\|paused\|exited\|dead

* 显示所有容器 -a 【包含exited 的容器】

\[ByCore@localhost ~\]$ sudo docker ps -a    \#STATUS 包含Exited 为 停止的容器，包含Up 为 运行的容器 后面 接 时间，表示运行或停止的时常

CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS                     PORTS                NAMES

f93de165c287   centos         "/bin/bash"              5 minutes ago   Exited \(0\) 5 minutes ago                        infallible\_shtern

79342bff2ffb   centos         "/bin/bash"              9 minutes ago   Exited \(0\) 7 minutes ago                        silly\_pike

23810f7ccd2a   6084105296a9   "/docker-entrypoint.…"   2 hours ago     Up 2 hours                 0.0.0.0:81-&gt;80/tcp   nginx-run

\[ByCore@localhost ~\]$

* 只显示 运行中的容器

\#方法一 ：docker ps  【不加任何参数】

\[ByCore@localhost ~\]$ sudo docker ps

CONTAINER ID   IMAGE          COMMAND                  CREATED       STATUS       PORTS                NAMES

23810f7ccd2a   6084105296a9   "/docker-entrypoint.…"   2 hours ago   Up 2 hours   0.0.0.0:81-&gt;80/tcp   nginx-run

\[ByCore@localhost ~\]$

\#方法二：sudo docker ps -a --filter=status=running

\[ByCore@localhost ~\]$ sudo docker ps -a --filter=status=running

CONTAINER ID   IMAGE          COMMAND                  CREATED       STATUS       PORTS                NAMES

23810f7ccd2a   6084105296a9   "/docker-entrypoint.…"   2 hours ago   Up 2 hours   0.0.0.0:81-&gt;80/tcp   nginx-run

\[ByCore@localhost ~\]$

* 只显示 停止状态的容器

\#方法一：docker ps --filter=status=exited

\[ByCore@localhost ~\]$ sudo docker ps --filter=status=exited

CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                      PORTS     NAMES

f93de165c287   centos    "/bin/bash"   15 minutes ago   Exited \(0\) 15 minutes ago             infallible\_shtern

79342bff2ffb   centos    "/bin/bash"   20 minutes ago   Exited \(0\) 18 minutes ago             silly\_pike

\[ByCore@localhost ~\]$

\#方法二：docker ps -a --filter=exited=0        \#筛选的是 STATUS 状态为 exited 且 信号 为 0

\[ByCore@localhost ~\]$ sudo docker ps -a --filter=exited=0

CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                      PORTS     NAMES

f93de165c287   centos    "/bin/bash"   29 minutes ago   Exited \(0\) 29 minutes ago             infallible\_shtern

79342bff2ffb   centos    "/bin/bash"   33 minutes ago   Exited \(0\) 31 minutes ago             silly\_pike

\[ByCore@localhost ~\]$

