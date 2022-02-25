# docker-lnmp
centos7下基于docker-compose搭建的lnmp简易版环境
`LNMP（Docker + Docker-compose + Nginx + MySQL5.7 + PHP8.1 + Redis5.0 + Memcached1.5 + Mongodb4.2）`

LNMP项目特点：
1. 一键安装，简单实用
2. 包含lnmp常用服务
3. 各服务支持数据文件、配置文件、日志文件挂载
4. 默认支持`pdo_mysql`、`mysqli`、`gd`、`curl`、`opcache`等常用扩展
5. 包含基本的已优化的配置文件

## 一. 目录结构

```php
├── data                        数据库数据目录
│   ├── mongo                   MongoDB 数据目录
│   ├── mysql                   MySQL 数据目录
│   └── redis                   Redis 数据目录
├── services                    服务构建文件、配置文件目录
│   ├── mysql                   MySQL 配置、构建文件
│   ├── nginx                   Nginx 配置、构建文件
│   ├── node                    Node 配置、构建文件
│   ├── php                     PHP 配置、构建文件
│   └── redis                   Redis 配置、构建文件
├── logs                        日志目录
│   └── mysql                   
│   └── nginx                   
│   └── php                         
│   └── redis             
├── docker-compose.yml                   Docker 默认服务配置文件，其它的可以 docker-composer -f 文件名
├── .env                                 环境配置
```
php-fpm容器中 **/usr/local/etc/** 目录结构
> 该目录下是php和php-fpm的配置文件，默认的结构如下，建议把整个目录挂载出来，方便修改和调试；<br />
> 建议：对于所有的配置文件的挂载都先生成默认文件、目录，然后再修改、调优；
```php
├── php                                          php配置目录
│   ├── php.ini-development                      php.ini开发环境
│   ├── php.ini-production                       php.ini生产环境
│   └── conf.d                                   php扩展配置目录
│       ├── docker-php-ext-redis.ini             redis扩展配置文件
│       ├── docker-php-ext-mongodb.ini           mongodb扩展配置文件 
│       └── ... 
├── php-fpm.conf                                 php-fpm配置文件
├── php-fpm.conf.default                         
├── php-fpm.d                                    php-fpm.conf可以包含该目录下的相关文件             
```
## 二. 快速使用
1. 本地安装
    1. `docker` 安装完成后，推荐使用阿里云`docker`加速：[https://help.aliyun.com/document_detail/60750.html](https://help.aliyun.com/document_detail/60750.html)
    2. `docker-compose` 可能会下载很慢，[这里附一个](https://pan.baidu.com/s/1ePbVGqjzN3nESOovg4-pKA)，提取码 `8o1r`
        1. 复制到 `/usr/local/bin`
        2. `chmod +x /usr/local/bin/docker-compose`
        3. `ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose`
        4. 测试安装：`docker-compose --version`

2. `clone`项目：
    $ `git clone https://github.com/ganyu/docker-lnmp-simple.git`

3. 如果不是`root`用户，还需将当前用户加入docker用户组：
    $ `sudo gpasswd -a ${USER} docker`

4. 可根据项目需要自行添加其它php扩展 

5. 启动本项目
    1. $ `docker network create --driver bridge lnmp-net` # 创建自定义bridge网络，这个**lnmp-net**在yml配置中用得到
    2. $ `docker-compose up --build --force-recreate`  #可以加 -d 后台运行，调试时不用加，方便查看日志；--force-recreate 这个参数也是为了调试方便，生产一定不用
    3. 启动后的三个基础容器显示：
    ```php
    [root@VM_0_13_centos ~]# docker ps -a
    CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS                      PORTS                                      NAMES
    3a0dd4e8968d        docker-lnmp_mysql     "docker-entrypoint.s…"   4 minutes ago       Up 4 minutes                33060/tcp, 0.0.0.0:6666->3306/tcp          mysql
    c8191d64d5a3        docker-lnmp_nginx     "nginx -g 'daemon of…"   4 minutes ago       Up 4 minutes                0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   nginx
    5450e694e90b        docker-lnmp_php-fpm   "docker-php-entrypoi…"   4 minutes ago       Up 4 minutes                0.0.0.0:9000-9001->9000-9001/tcp           php-fpm
    1bdfb82a502b        51e17a6261bd          "/bin/sh -c 'mkdir -…"   10 months ago       Exited (56) 10 months ago                                              hopeful_rosalind
    ```

## 三. docker常用命令
- $ `systemctl start docker`    # 启动docker
- $ `docker start containername` # 启动容器
- $ `docker stop containername` # 停止容器
- $ `docker exec -it 13a676bb9bff /bin/bash` # 进入容器
- $ `docker rm containername` # 删除容器
- $ `docker rmi image` # 删除镜像

- $ `docker-compose up` # 启动所有容器 
- $ `docker-compose up nginx php mysql` # 启动指定容器
- $ `docker-compose up -d nginx php mysql` # 后台运行方式启动指定容器 
- $ `docker-compose up --build --force-recreate` # 强制启动
- $ `docker-compose start php` # 启动服务
- $ `docker-compose stop php` # 停止服务
- $ `docker-compose restart php` # 重启服务
- $ `docker-compose build php` # 使用Dockerfile构建服务

## 四. docker网络模式
> docker 网络模式是学习docker不可或缺的一部分，搞懂这块才能轻松应对容器间的连接
+ 研读官方文档 [https://docs.docker.com/network/bridge](https://docs.docker.com/network/bridge)
+ 常用命令
    + $ `docker network ls` # bridge none host 三种模式
    + $ `docker network create --driver bridge lnmp-net` # **创建自定义网络**
    + $ `docker network connect lnmp-net 容器ID` # 给容器设置网络，使用docker-compose是无需这样单独设置的，直接在yml中指定networks即可
    + $ `docker network inspect lnmp-net` # 查看网络详情，显示哪些容器加入了该网络
    + $ `docker inspect --format='{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)` #查看为每个容器分配的ip
+ bridge模式
> bridge是默认模式也是最常用的模式，官方建议使用自定义bridge网络
1. bridge模式下可以通过**容器名称进行容器间的通信**
2. 多个容器可以使用同一个`docker-compose.yml`是**无需使用--link**来连接的
3. 本实例使用都是自定义bridge，关于docker网络模式更多知识，自行查阅文档
     
## 五. Mysql基本操作
+ 远程连接需要进入容器登录mysql授权
    + $ `docker exec -it mysql /bin/bash`
    + $ `mysql -uroot -p123456`
    + $ `select host,user,plugin,authentication_string from mysql.user;` # 查看有无 root对应 %
    + $ `update user set host ='％' where user ='root' and host ='localhost';` # 建议这样
    + $ `grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;` # 与update二选一
    + $ `flush privileges;`
  
+ mysql新建一个用户用于远程连接和操作指定库
    + 创建用户：`create user 'yii'@'%' identified by '123456';`
    + 授权：`grant all privileges on hshop.* to "yii"@"%" identified by "123456";`
    + 刷新：`flush privileges;`
    + 端口：`是指在.env中配置的对外端口，这个和上面'db'连接数据库不同`

## 六. nginx配置说明
+ 默认的nginx配置已支持yii2的运行
+ 如项目放在 `/var/www/order`
    + .env中，`WEB_DIR=/var/www/order`
    + nginx.conf中， `root /var/www/order/backend/web;`
+ nginx和php-fpm必须挂载相同的项目目录，否则会导致静态文件可以访问，php无法访问

<!--## 问题
### 如何使用容器内的php环境来执行主机中（项目一般会挂载在主机）的php脚本？
1. 参考官网：[https://hub.docker.com/_/php/](https://hub.docker.com/_/php/)
可以使用php-cli，但这仅仅是一个php-cli环境，比如使用它来执行yii中的命令行，就会提示找不到数据库，还需要配置一个和php-fpm相同的环境？？？
-->

## 增加端口映射
https://zhuanlan.zhihu.com/p/65938559

### docker中安装vim
apt-get update

apt-get install -y vim

## 官方文档
1. [https://docs.docker.com/compose/compose-file/](https://docs.docker.com/compose/compose-file/) # `docker-compose.yml`规范文档

### 如有错误敬请指正（issues）
