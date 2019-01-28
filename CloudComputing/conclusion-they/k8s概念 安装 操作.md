### Kubernetes介绍
>是一个全新得基于容器得分布式架构领先方案，是Google开源的容器集群管理系统

>Kubernetes是一个完备的分布式系统支撑平台，具有完备的集群管理能力，多扩多层次的安全防护和准入机制、多租户应用支撑能力、透明的服务注册和发现机制、內建智能负载均衡器、强大的故障发现和自我修复能力、服务滚动升级和在线扩容能力、可扩展的资源自动调度机制以及多粒度的资源配额管理能力。





### Kubernetes工具
##### Service
>Service是分布式集群架构的核心，一个Service对象拥有如下关键特征
- 拥有一个唯一指定的名字
- 拥有一个虚拟IP（Cluster IP、Service IP、或VIP）和端口号
- 能够体统某种远程服务能力
- 被映射到了提供这种服务能力的一组容器应用上
##### Replication Contrpller（RC）新版本被deployment取代
>帮助解决ID系统中服务扩容和升级的两大难题，你只需为需要扩容的Service关联的Pod创建一个Replication Controller简称（RC），则该Service的扩容和后续的升级都得到解决。

>一个rc有3个关键信息
- 目标Pod的定义
- 目标Pod需要运行的副本数量（Replicas）
- 要监控的目标Pod标签（Label）

#### Kubernetes优势
- 容器编排
- 轻量级
- 开源
- 弹性伸缩
- 负载均衡

#### Kubernetes架构和组件
![image](4145CB58959E41D08B2EDD778C940CDC)
### 搭建Kubernetes
>这里使用压缩包中的shell脚本一键安装。
* master节点
```
sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/selinux/config
systemctl disable iptables-services 
systemctl stop iptables-services 
yum install epel-release
docker info #docker版本不能过低
systemctl start docker
vi /etc/hosts  #添加master和node的IP和name
 tar -xvf 1kubernetes1.9.2.tar.tar.gz
cd shell
sh init.sh #可能没有第七行的命令，可以注释掉
sh master.sh
kubectl get pod -n kube-system #得到的所有状态STATUS全部为Running
kubectl get node #查看节点
```
* node节点（多个节点相同命令）

```
vi /etc/hosts
docker info
systemctl start docker
tar -xvf 1kubernetes1.9.2.tar.tar.gz 
sed -i.bak 's/SELINUX=enforcing/SELINUX=permissive/' /etc/selinux/config
cd shell
sh init.sh #可能没有第七行的命令，可以注释掉
#这里执行的命令是master节点执行master.sh之后得到的
kubeadm join --token a6cd73.608af4b6b8c67013 192.168.233.167:6443 --discovery-token-ca-cert-hash sha256:81f0ab9f22ae362b2dcd6d4cc383a5f1e7fb6d2edaff32842f81b2620383bc92
```
>https:/master_IP:32000打开dashboard的web网页，使用火狐浏览器添加例外证书进行访问



#### 集群管理
* 查看集群信息
```
kubectl cluster-info
```
* GET信息
```
kubectl get nodes
kubectl get pods
kubectl get rc,namespace，svc(service)
kubectl get pod pod_name -o wide #查看pod被分配在哪个node上
```
* describe
>describe类似于get，同样用于获取resource的相关信息。不同的是，get获得的是更详细的resource个性的详细信息，describe获得的是resource集群相关的信息。describe命令同get类似，但是describe不支持-o选项，对于同一类型resource，describe输出的信息格式，内容域相同。
注：如果发现是查询某个resource的信息，使用get命令能够获取更加详尽的信息。但是如果想要查询某个resource的状态，如某个pod并不是在running状态，这时需要获取更详尽的状态信息时，就应该使用describe命令。

```
kubectl describe pod pod_name
```
* 创建pod

```
kubectl create -f filename.yaml
#kubectl run --image=镜像 命名空间 --port=80
```
* replace更新替换资源
>replace命令用于对已有资源进行更新、替换。如前面create中创建的nginx，当我们需要更新resource的一些属性的时候，如果修改副本数量，增加、修改label，更改image版本，修改端口等。都可以直接修改原yaml文件，然后执行replace命令。
注：名字不能被更更新。另外，如果是更新label，原有标签的pod将会与更新label后的rc断开联系，有新label的rc将会创建指定副本数的新的pod，但是默认并不会删除原来的pod。所以此时如果使用get pod将会发现pod数翻倍，进一步check会发现原来的pod已经不会被新rc控制.

```
kubectl replace -f rc-nginx.yaml
```
* patch在容器运行过程中修改
>前面创建pod的label是app=nginx-2，修改为app=nginx-3
```
kubectl patch pod rc-nginx-2-kpiqt -p '{"metadata":{"labels":{"app":"nginx-3"}}}'
```
* edit(更新)
>使用edit直接更新前面创建的pod

```
kubectl edit pod rc-nginx-btv4j
```
>效果等于

```
kubectl get pod rc-nginx-btv4j -o yaml >> /tmp/nginx-tmp.yaml #以yaml格式重定向到文件中
vim /tmp/nginx-tmp.yaml
/*do some changes here */ #进行修改
kubectl replace -f /tmp/nginx-tmp.yaml #使用replace更新替换资源
```
* Delete删除
```
kubectl delete -f rc-nginx.yaml

kubectl delete pod rc-nginx-btv4j
kubectl delete pod -lapp=nginx-2
```
>使用run创建的pod直接使用delete+podname删除不掉，删除之后由于高可用的原因会自动默认为需要自动添加一个。所以需要删除deployments

```
kubectl get deployments
kubectl delete deployments name
kubectl delete pod pod_name
```

* 扩展

```
kubectl get deployments
kubectl scale --replicas=3 deployments/name
```
* 滚动升级

```
kubectl get deployments
kubectl set images deployments/name name=newname
```
##### 问题
* 端口问题，拒绝访问
>The connection to the server localhost:8080 was refused - did you specify the right host or port?

```
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
. ~/.bash_profile
```

* 安装有问题，卸载删除

```
kubeadm reset
rm -rf /var/etcd
rm -rf /var/lib/etcd
```



### yaml创建pod


```
apiVersion: v1 #指定api版本，此值必须在kubectl apiversion中  
kind: Pod #指定创建资源的角色/类型  
metadata: #资源的元数据/属性  
  name: web04-pod #资源的名字，在同一个namespace中必须唯一  
  labels: #设定资源的标签，详情请见http://blog.csdn.net/liyingke112/article/details/77482384
    k8s-app: apache  
    version: v1  
    kubernetes.io/cluster-service: "true"  
  annotations:            #自定义注解列表  
    - name: String        #自定义注解名字  
spec:#specification of the resource content 指定该资源的内容  
  restartPolicy: Always #表明该容器一直运行，默认k8s的策略，在此容器退出后，会立即创建一个相同的容器  
  nodeSelector:     #节点选择，先给主机打标签kubectl label nodes kube-node1 zone=node1  
    zone: node1  
  containers:  
  - name: web04-pod #容器的名字  
    image: web:apache #容器使用的镜像地址  
    imagePullPolicy: Never #三个选择Always、Never、IfNotPresent，每次启动时检查和更新（从registery）images的策略，
                           # Always，每次都检查
                           # Never，每次都不检查（不管本地是否有）
                           # IfNotPresent，如果本地有就不检查，如果没有就拉取
    command: ['sh'] #启动容器的运行命令，将覆盖容器中的Entrypoint,对应Dockefile中的ENTRYPOINT  
    args: ["$(str)"] #启动容器的命令参数，对应Dockerfile中CMD参数  
    env: #指定容器中的环境变量  
    - name: str #变量的名字  
      value: "/etc/run.sh" #变量的值  
    resources: #资源管理，请求请见http://blog.csdn.net/liyingke112/article/details/77452630
      requests: #容器运行时，最低资源需求，也就是说最少需要多少资源容器才能正常运行  
        cpu: 0.1 #CPU资源（核数），两种方式，浮点数或者是整数+m，0.1=100m，最少值为0.001核（1m）
        memory: 32Mi #内存使用量  
      limits: #资源限制  
        cpu: 0.5  
        memory: 32Mi  
    ports:  
    - containerPort: 80 #容器开发对外的端口
      name: httpd  #名称
      protocol: TCP  
    livenessProbe: #pod内容器健康检查的设置，详情请见http://blog.csdn.net/liyingke112/article/details/77531584
      httpGet: #通过httpget检查健康，返回200-399之间，则认为容器正常  
        path: / #URI地址  
        port: 80  
        #host: 127.0.0.1 #主机地址  
        scheme: HTTP  
      initialDelaySeconds: 180 #表明第一次检测在容器启动后多长时间后开始  
      timeoutSeconds: 5 #检测的超时时间  
      periodSeconds: 15  #检查间隔时间  
      #也可以用这种方法  
      #exec: 执行命令的方法进行监测，如果其退出码不为0，则认为容器正常  
      #  command:  
      #    - cat  
      #    - /tmp/health  
      #也可以用这种方法  
      #tcpSocket: //通过tcpSocket检查健康   
      #  port: number   
    lifecycle: #生命周期管理  
      postStart: #容器运行之前运行的任务  
        exec:  
          command:  
            - 'sh'  
            - 'yum upgrade -y'  
      preStop:#容器关闭之前运行的任务  
        exec:  
          command: ['service httpd stop']  
    volumeMounts:  #详情请见http://blog.csdn.net/liyingke112/article/details/76577520
    - name: volume #挂载设备的名字，与volumes[*].name 需要对应    
      mountPath: /data #挂载到容器的某个路径下  
      readOnly: True  
  volumes: #定义一组挂载设备  
  - name: volume #定义一个挂载设备的名字  
    #meptyDir: {}  
    hostPath:  
      path: /opt #挂载设备类型为hostPath，路径为宿主机下的/opt,这里设备类型支持很多种  

```


