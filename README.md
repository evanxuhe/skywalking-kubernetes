[![GitHub stars](https://img.shields.io/github/stars/evanxuhe/skywalking-kubernetes.svg?style=for-the-badge&label=Stars&logo=github)](https://github.com/evanxuhe/skywalking-kubernetes)

[![](https://images.microbadger.com/badges/image/evanxuhe/skywalking-oap-server.svg)](https://microbadger.com/images/evanxuhe/skywalking-oap-server "Get your own image badge on microbadger.com")
[![](https://images.microbadger.com/badges/image/evanxuhe/skywalking-agent-sidecar:6.1.0.svg)](https://microbadger.com/images/evanxuhe/skywalking-agent-sidecar:6.1.0 "Get your own image badge on microbadger.com")
[![](https://images.microbadger.com/badges/image/evanxuhe/apm-eureka.svg)](https://microbadger.com/images/evanxuhe/apm-eureka "Get your own image badge on microbadger.com")
[![](https://images.microbadger.com/badges/version/evanxuhe/skywalking-agent-sidecar:6.1.0.svg)](https://microbadger.com/images/evanxuhe/skywalking-agent-sidecar:6.1.0 "Get your own version badge on microbadger.com")
# skywalking-kubernetes
该项目可以迅速将skywalking 6.1.0部署进kubernetes(k8s) 

包含ui oap es模块和完整的springcloud测试用例

此外将agent整合到sidecar中，也就是说每个pod中有两个应用 app+agent sidecar，更加适合于生产环境
## 描述
我弄这个主要是为了学习整合skywalking作为kubernetes线下环境的APM

但是skywalking官方[apache/skywalking-kubernetes](https://github.com/apache/skywalking-kubernetes)的一些配置在apache孵化后过期
没有最新的6.1.0版本 而6.1.0的性能提升比较大，此外也支持Elasticsearch6.3.2 非常值得学习。
此外一些配置面向云环境，不适合本地开发测试

-------------
# 安装使用
**git地址**
[evanxuhe/skywalking-kubernetes](https://github.com/evanxuhe/skywalking-kubernetes)
    cd 6.1.0

    kubectl apply -f namespace.yml
    kubectl apply -f elasticsearch
    kubectl apply -f oap
    kubectl apply -f ui
    kubectl apply -f apm-springcloud-demo
**注意修改elasticsearch/01-pv.yml中的路径为本地路径;节点名为本机hostname** 

浏览器打开[http://127.0.0.1:31234/](http://127.0.0.1:31234/)查看服务详情

如果部署了测试用例 访问[http://127.0.0.1:30082/item](http://127.0.0.1:30082/item) 会发现UI拓扑图多了两个服务 如下

# 成果展示
```
NAME                            READY   STATUS    RESTARTS   AGE
apm-eureka-799b8d5449-72npx     1/1     Running   0          99m
apm-item-bdf545d9c-kjqxd        1/1     Running   0          94m
elasticsearch-0                 1/1     Running   0          119m
oap-6b56f8bbf5-7fjvb            1/1     Running   0          118m
oap-6b56f8bbf5-7tldw            1/1     Running   0          118m
oap-6b56f8bbf5-qzdx2            1/1     Running   0          118m
ui-deployment-f4799496c-m5xw6   1/1     Running   0          117m
```
![avatar](https://img-blog.csdnimg.cn/2019052320472034.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2V2YW54dWhl,size_16,color_FFFFFF,t_70)
![avatar](https://img-blog.csdnimg.cn/20190523204652721.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2V2YW54dWhl,size_16,color_FFFFFF,t_70)

# 模块概述
### 模块概述
|component  | descripiton |
|--|--|
| namespace |创建skywalking命名空间|
| elasticsearch |映射本地磁盘卷的es集群|
| oap |collector 收集agent上传的数据并整合|
| ui | RocketUI展示前端数据 |
| apm-springcloud-demo | springcloud应用demo，eureka+item service 产生数据供展示(可选) |
| busybox.yml | 装了很多调试工具，比如net-tools等，方便定位k8s内问题|
### 版本描述
|component  | version |
|--|--|
| kubernetes |1.14.2|
| docker| 18.06 |
| skywalking |6.1.0|
| elasticsearch | 6.4.0 |


### 镜像
|image  | version | descripiton |
|--|--|--|
| evanxuhe/skywalking-oap-server |6.1.0|修正时区为东八区|
| evanxuhe/skywalking-agent-sidecar | 6.1.0 |sidecar 里面装载了agent文件夹|
| apache/skywalking-ui|6.1.0|官方ui镜像|
|elasticsearch-oss|6.3.2|官方es镜像|

# Ids can't be null
问题的根源在于**时区不匹配** OAP为UTC UI为UTC+8 所以UI在对应的时间区间内查不到service导致报错
解决办法
在base镜像中 添加
```bash
# 时区修改为东八区
RUN apk add --no-cache tzdata
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
```
# 使用说明

>应用全部挂在在skywalking namepace下，所以大家使用时不要忘记切换namespace 比如加-n skywalking

> 此外之前遇到过很奇怪的问题 oap可以正常收集数据，ui没显示，并且请求oap报错IDs can't be null其实就是pod时间不对，导致查询范围不匹配
查看kubectl get pods -n skywalking
## 配置修改
es使用statefulset方式部署，oap，ui使用deployment部署
因而想要修改副本数，内存，磁盘等请修改对应目录下的03-statefulset.yml或者03-deployment.yml
## 服务起停
由于k8s deployment会在pod停止后一直重启
因而修改停止的正确做法是 

    kubectl edit statefulset elasticsearch -n skywalking
    kubectl edit deployment oap -n skywalking
    kubectl edit deployment ui-deployment -n skywalking

修改对应的replicates为0，拓展应用将replicates修改为对应副本数即可，想刷新配置也可以用这种办法

----
# 修改内容
基于skywalking官方[apache/skywalking-kubernetes](https://github.com/apache/skywalking-kubernetes)
修改为：

 - oap，ui镜像更新为最新 apache/skywalking-oap-server,apache/skywalking-ui镜像，解决原有镜像过期问题,修改时区为东八区
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
欢迎大家查看[我的博客](https://blog.csdn.net/evanxuhe/article/details/90211950)评论交流

由于本人是k8s新人，难免存在很多问题，欢迎大家反馈
我的邮箱 xuhe@chehejia.com
