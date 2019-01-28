## 准备工作
1. 两台虚拟机 都要有docker
1. 第一台做主机 需要有nginx镜像
1. 都开启docker
```
docker-compose安装
yum -y install epel-release
yum -y install python-pip
pip install docker-compose
```

2. 第一台安装  yum -y install docker-compose
3. 查看网卡信息 docker network ls
## 具体操作

#### 1执行 docker swarm init
获得一串如下
- docker swarm join 
- token SWMTKN-1-49et5nixvg2xjtechw4s4ozla8zkvp4z19bwjn5ug2knhvqkd7-drxqbc3vhmlxo1lqay2yh22yq \
-  192.168.226.156:2377

#### 2拿到第二台虚拟机里运行这串
![image](E70451A017674C48BBD09E0332EAF9E2)
#### 3回到第一台执行

```
docker node ls
```
就会发现有node列了
![image](2D2AC94CA3DA4B6D9F0C40236DE8BB75)
#### 创建两个附属的

```
docker service create --replicas 2 -p 8090:80 --name myfirst_ds nginx
```
- --replicas 附属的数量
- -p  端口号
- --name 服务的名字

#### 扩充附属的数量(比如扩充到五个)

```
docker service scale myfirst_ds=5
```
##### 查看服务的信息

```
docker service ls
```
##### 查看运行的容器

```
docker ps
```


### 给服务用的镜像在线更新(比如修改它的index.html)

##### 1把它使用的image run起来

```
docker run -p 140:80 --name newimage1 -d new_nginx_image
```


##### 2先写一个新的index.html
(我是放在/usr/docker/nginx/html中的)
![image](3A3877EC9A2B4F179523B8757A7752A1)
##### 3用这个去替换掉run起来的这个容器的index.html

```
docker cp  /usr/docker/nginx/html/index.html 40a607932ffb:/usr/share/nginx/html/
```

##### 4把这个run着的容器做成image

```
docker commit 40a607932ffb new_nginx_image1
```

##### 5更新service使用的image
docker service update --image new_nginx_image1 myfirst_ds
查看更新情况

```
docker service ps myfirst_ds
```

##### 6稍等一会儿去网页上查看即会更新

![image](C07316F4BB7842EC8DB6BD96D8D69019)

