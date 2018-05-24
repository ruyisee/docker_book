## docker命令速查
* 轻量级操作系统虚拟化解决方案
* 基于Linux容器(LXC)等技术的进一步封装
* docker容器在操作系统层面实现虚拟化, 传统方式则是硬件层面实现

命令 | 意义 
---|---
docker login $hub-server-host$ | 登录远程hub服务器
docker search $searchStr$ | 搜索仓库
docker images | 查看本地所有镜像
---|---
docker run -d -p $hostPort$ : $containerPort$ -v $hostPath$:$containerPath$ $image$ | 运行镜像, 得到一个容器, -c可以限制cpu使用量
--volume-from $dataContainer$ | 挂载数据容器, 数据卷并不需要保持运行状态
-P(大写) | 随机选取主机的49000-49900映射到主机的开放端口(5000)
-p | 可以使用多次用来绑定多个特定端口
-p $hostPort:contianerPort/udp$ | 指定udp 端口
--link name:alias | 容器互联技术, name是链接容器的名称, alias是这个链接的别名
---|---
docker cp $hostPath$  $containerPath | 复制文件到容器
docker stop $containeridORname | 停止某个容器
docker restart $containderidORname | 重启某个容器
docker start $containeridORname | 启动已经停止的容器
docker port $containeridORname$ | 查看容器的端口映射
docker inspect $containeridORname$ | 查看容器的详细信息
docker logs $containeridORname$ | 获取容器的log信息
docker tag $imageid$ $name$:$tag$ | 更改image名称和tag
docker exec -it $container$ bash | 交互模式进入运行中的容器, exit 退出
docker attach $containeridORname$ | 进入容器, 多个attach会阻塞,同步显示,不推荐.
docker rmi -f $containerid$ | 删除容器
docker rm -f $imageid$ | 删除镜像-此镜像创建出的容器也将被强制删除.
docker run $containerid$ yum install ... | 修改容器
docker commit $containerid$  $iamgename/tag$ | 保存已经修改的容器为镜像
docker ps | 查看运行中的容器 -a查看全部, -l查看最近一次运行的容器  
docker creat -v $hostPath/containerPath$ --naem $containername$ $baseImage$ | 创建数据卷
docker rmi -v | 删除最后一个挂载数据卷的容器的时候, 指定要删除的挂载数据卷
docker save $imageid$ > $filename$| 打包镜像, 保留完整记录
docker load < $filename$ | 还原镜像 
docker export _containerid_ > _filename_ | 打包导出容器 ,将丢弃所有的历史记录和元数据信息,仅仅保留容器当时的快照状态
docker import < _filename_ | 还原容器

## 重要功能

### 容器互联技术

* 在两个互联的容器之间创建一个安全隧道, 不会映射端口到宿主机上
* 可以通过 **主容器** 的 **环境变量** OR /etc/hosts 看到容器公开信息
* 可以发现, 其实就是通过**docker0**虚拟网桥构建的局域网
```shell
# 链接方式 --link name:alias |  name是链接容器的名称, alias是这个链接的别名
# 查看住容器的环境变量
$ docker run --rm --name web2 --link db:db training/webapp env ...
DB_NAME=/web2/db # 能够看到 alias(大写)_NAME=/containernaem/alias
DB_PORT=tcp://172.17.0.5:5432 DB_PORT_5000_TCP=tcp://172.17.0.5:5432
DB_PORT_5000_TCP_PROTO=tcp DB_PORT_5000_TCP_PORT=5432 DB_PORT_5000_TCP_ADDR=172.17.0.5 ...
# 另一种方式, 查看主容器的/etc/hosts
 $ sudo docker run -t -i --rm --link db:db training/webapp /bin/bash root@aed84ee21bde:/opt/webapp# cat /etc/hosts
172.17.0.7 aed84ee21bde # 主容器
...
172.17.0.5  db # 链接容器
```