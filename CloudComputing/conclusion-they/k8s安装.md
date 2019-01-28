## 准备工作 
- 两台虚拟机安装最新版本的docker并开启
- root目录下 都上传1kubernetes1.9.2.tar.tar.gz
- 分别修改好主机名 
master和note

```
hostnamectl set-hostname master
hostnamectl set-hostname node1
```
- 修改配置文件
> vi /etc/hosts

```
192.168.226.171  master
192.168.226.173   node1
```



## master主机
关闭防火墙和selinux
```
setenforce 0
systemctl disable  firewalld
systemctl stop  firewalld
```

```
mkdir /root/k8s1
cp 1kubernetes1.9.2.tar.tar.gz  /root/k8s1
swapoff -a
sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/selinux/config
yum -y install epel-release
```
解压并执行里面的脚本
```
cd /root/k8s1
tar -zxvf 1kubernetes1.9.2.tar.tar.gz
cd shell
sh init.sh
sh master.sh
```
执行 sh master.sh会得到一串如下字串 

```
kubeadm join --token 7a4339.9cd2f419faa42db6 192.168.226.171:6443 --discovery-token-ca-cert-hash sha256:9278708342d142afbbceb67be8750a4cc9697529de694d180655c95c034808b1
```


验证
kubectl get pod -n kube-system

![image](BE0248A0CE7E42B9916104259E23BF12)

## note主机
关闭防火墙和selinux
```
setenforce 0
systemctl disable  firewalld
systemctl stop  firewalld
```
命令
```
mkdir/root/k8s2
cp 1kubernetes1.9.2.tar.tar.gz  /root/k8s1
swapoff -a
sed -i.bak 's/SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config
```
解压并执行里面的脚本
```
cd /root/k8s2
tar -zxvf 1kubernetes1.9.2.tar.tar.gz
cd shell
sh init.sh
```
执行master 上执行master.sh的那串即可

### 验证 
在master上执行kubectl get node
![image](E0A530BFBC3B4163B992E28588B9E155)

打开火狐浏览器
https://192.168.226.171:32000



