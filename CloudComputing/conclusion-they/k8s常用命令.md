# 查
查看集群信息
>kubectl cluster-info
![image](583BF9B0E4324E6B948FDAF5C34CA237)

查看各组件状态
>kubectl -s https://192.168.226.171:6443 get componentstatuses
![image](12176CF8848C47EF931FE11521AF3E41)

查看所有的nodes(节点)具体信息
>kubectl get nodes -o wide
![image](10EA8BCDF900481B8EAEAEA857715046)

查看所有的node
>kubectl get nodes
![image](F51E396580BA40C5A197AA8D71C7532C)

查看rc和namespace(命名空间)
>kubectl get rc,namespace
![image](3B13F10057A84D5FABABCEE1E0CBA7F4)

查看所有pod和svc(和service一样)
>kubectl get pods,svc
![image](3CA5743BC3EC4ADDA48E62EB7AC0C32E)

以json格式输出pod的详细信息(太长不截图了)
>kubectl get pod nginx-app-66c69c5fb9-8hj96 -o json

查看指定pod跑在哪个node上
> kubectl get po nginx -o wide
![image](A9394EF82EB9441AAA134133CEE5E1A2)

查看所有的pods泡在哪个node上
>kubectl get pods -o wide
![image](6E9BD3E02D294D319215476D18805864)

查看部署情况
>kubectl get deployments
![image](985B23BC59D4417EAB179F723B1B1F3C)


查看日志
>kubelet logs pod redis-master-jwbss


# 加
创建podrun起来
>kubectl run --images=nginx:1.9.1 nginx-ap --port=80

扩展
>kubectl scale --replicas=3 deployment/nginx-app 

滚动升级
>kubectl set image deployment/nginx-app nginx-app=nginx

yaml
我们使用yaml是因为他像XML或Json 一样利于读写的一种数据格式

yaml基本规则
1. 大小写敏感
1. 使用缩进表示层级关系
1. 禁止使用TAB缩进,只能是空格
1. 缩进没有要求,只要对齐就表示一个层级
1. #表示注释
1. 字符串也可不用引号标注

数据类型
对象(map)








