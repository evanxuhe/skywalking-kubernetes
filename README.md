# skywalking-kubernetes
轻松将skywalking 6.0部署进kubernetes(k8s) 
完整ui oap es模块 可以基于k8s快速构建起一个完整本地skywalking本地测试开发环境
## 适用范围
我弄这个主要是为了本地测试k8s 搭建skywalking 学习了解之后逐步迁移到生产环境
但是skywalking官方[apache/skywalking-kubernetes](https://github.com/apache/skywalking-kubernetes)的一些配置在apache孵化后过期，一些配置面向云环境，不适合本地测试
该代码在本地开发机测试(ubuntu 18.04TLS)正常
## 版本说明
|component  | version |
|--|--|
| minikube | 1.0.1 |
| kubernetes |1.14.1|
| skywalking |6.0.0-GA|
| elasticsearch | 6.4.0 |


本文写于**2019/05/14，
skywalking用的是较新的6.0.0-GA，架构性能变化较5.0变化较大，有很大的参考意义；kubernetes为最新版
elasticsearch使用6.4.0较稳定版本，与skywalking兼容性较好，亲测7.0有较大问题

-------------
# 安装使用
**git地址**
[evanxuhe/skywalking-kubernetes](https://github.com/evanxuhe/skywalking-kubernetes)

clone下来，依次部署即可，非常简单方便
**注意修改elasticsearch/01-pv.yml中的路径为本地路径**

    kubectl apply -f namespace.yml
    kubectl apply -f elasticsearch
    kubectl apply -f oap
    kubectl apply -f ui

应用全部挂在在skywalking namepace下，所以大家使用时不要忘记制定-n skywalking
查看kubectl get pods -n skywalking
## 配置修改
es使用statefulset方式部署，oap，ui使用deployment部署
因而想要修改副本数，内存，磁盘等请修改对应目录下的03-statefulset.yml或者03-deployment.yml
## 服务起停
由于k8s deployment会在pod停止后一直重启
因而修改停止的正确做法是 

    kubectl edit statefulset elasticsearch -n skywalking
    kubectl edit deployment oap-deployment -n skywalking
    kubectl edit deployment ui-deployment -n skywalking

修改对应的replicates为0，拓展应用将replicates修改为对应副本数即可，想刷新配置也可以用这种办法

----
# 修改内容
基于skywalking官方[apache/skywalking-kubernetes](https://github.com/apache/skywalking-kubernetes)
修改为：

 - oap，ui镜像更新为最新 apache/skywalking-oap-server,apache/skywalking-ui镜像，解决原有镜像过期问题
 - elasticsearch存储卷配置gce修改为local,增加01-pv.yaml 大家需要制定自己的本地路径
 - ui负载均衡从云环境loadbalace改为适用于本地的nodeport
---
# 故障排除
**显示pod不存在**
kubectl config set-context $(kubectl config current-context)  --namespace=skywalking
或者在所有命令后指定namespace  如 kubectl get pods -n skywalking
**服务出现Error**
查看event是否有错误信息

    kubectl describe pod [pod名]

 
如果没有，说明进程已经挂掉，查看日志是否有错误信息

    kubectl log [pod名]
**dns域名解析错误**
遇到一个问题，es一直正常运行，但是oap报错找不到es，这是由于域名解析问题
k8s登录elasticsearch，查看hostname ip地址
登录另一个可用pod，如ui 看是否能ping 通ip(网络问题)，能否ping通elasticsearch主机名(dns问题)
最后发现是coredns服务没能正常启动，导致所有dns不用这个错误很常见。。。

    kubectl get pods -n kube-system
解决方案详见[coredns服务一直处于 CrashLoopBackOff状态]( https://blog.csdn.net/evanxuhe/article/details/90210764)
# 联系方式
由于本人是k8s新人，难免存在很多问题，欢迎大家反馈
我的邮箱 xuhe@chehejia.com