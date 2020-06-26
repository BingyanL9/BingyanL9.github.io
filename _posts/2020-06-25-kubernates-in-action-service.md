---
layout: post
title:  "K8s - Service(-) "
date:   2020-06-25 21:18:54
categories: K8s Service(-)
tags: K8s Service Foundation
excerpt: 服务：让客户端发现pod并与之通信
mathjax: true
---

* content
{:toc}

> Services是kubernete的系统资源， 为一组功能相同的pods提供单一的固定的接口。在服务存在期间，它的ip和端口不会改变。在它托管的pods中进行负载均衡。也可对外部资源进行负载均衡和服务发现。

### **服务介绍**

Kubernates 服务是一种为一组功能相同的 pod 提供单一不变的接入点的资源。当服务存在时，它的 IP 地址和端口不会变。客户端通过 IP 地址和端口号建立连接，这些连接会被路由到该服务的任意一个 pod 上。正因为这种方式，客户端不需要知道每一个单独的 pod 的地址，pod可以被随时创建了移除。

- 服务的主要目标是使集群内的其他 pod 可以访问当前这组pod，但通常也希望对外暴露服务

- 服务表示一组或者多组提供相同服务的 pod 的静态地址

- LoadBalancer: 创建一个外部负载均衡，可以通过外部的负载均衡的公共 IP 访问pod

### **创建服务**

1. kubectl expose

    ```
    kubectl expose rc [ReplicationController] --type=LoadBalancer --name [service-name]
    ```

2. Yaml

    创建一个名字叫做 kube-svc 的服务，他将在端口80接受请求并连接路由到具有标签选择器是 app=kubia 的 pod 的8080的端口上

    ```
    apiVersion: v1
    kind: Service
    metadata:
      name: kube-svc
    spec:
      ports:
      - port: 80 ## 该服务的可用端口
        targetPort: 8080 ## 服务将连接转发到的容器端口
      selector:
        app: kubia ## 具有app=kubia标签的pod都属于该服务
    ```

    创建命令：

    ```
    kubectl create -f *.yaml
    or
    kubectl apply -f *.yaml
    ```

3. 执行完创建命令后，检测新的服务并测试可用性

    ```
    kubectl get svc -n [your-namespace]
    or
    kubectl get service -n [your-namespace]
    ```

    查询结果如下，发现已经分配了内部集群 IP， 但是并未对外暴露

    ![](\images\service\get-service-result.png)

    从内部集群测试服务：

    ```
    kubectl exec [pod-name] -- command 
    eg: kubuctl exec kubia-gpu -- curl -s http://172.21.252.138:80
	##-- 两个横杆之后的内容是指在pod内部需要执行的命令。
    ```

    ![](\images\service\ping-result.png)

    可以看到每次ping响应的pod不一定，service 在做负载均衡和资源分配。

4. 配置服务上的会话亲和性

    如果多次执行相同的命令，就是 curl 去访问某个pod。每次调用执行应该在不同的pod上。因为服务代理通常将每个连接随机指向选中的后端pod的一个，即使连接来自同一个客户端。那么怎么样才能够使特定的客户端产生的所有请求都指向同一个 pod 呢？可以设置服务的 sessionAffinity 属性为 clientIP,默认为 none。

    ![](\images\service\sessionAffinity.png)

    像这样！
    这种方式将会使服务代理将来自同一个 clintIP 的所有请求转发至同一个pod上。

5. 同一个服务暴露多个端口

    HTTP 监听一个端口；HTTPS 监听另外一个端口。可以使用一个服务从端口80和443转发至pod端口的8080和8443。 但是在创建一个有多端口的服务时，必须给每个端口指定名字。

    yaml:

    ```
    apiVerision: v1
    kind: service
    metadata: 
    name: kubia
    spec: 
    ports:
    - name: http
      port: 8080
      targetPort: 8080
    
    - name: https
      port: 443
      targetPort: 8443
    selector:
        app: kubia

    ```
    **Notes:** 标签选择器应用于整个服务，不能对单个端口做单独配置。如果不同的 pod 有不同的映射关系，需要创建两个服务。

6. 使用命名的端口

    通常情况下我们通过数字来指定端口，但是在服务spec中也可以直接使用pod中命名好的端口。这样对于一些不是众所周知的端口号，使得服务的 spec 更加清晰。

    pod yaml:

    ![](\images\service\naming-port-pod.png)

    svc yaml:

    ![](\images\service\naming-port-svc.png)

    **好处：**可以更换pod的端口而无需更改service
    
### **服务发现**

客户端pod如何发现服务的ip和端口呢？ kubernetes为客户端提供了发现服务的ip和端口的方式

1. 通过环境变量发现服务

    在 pod 开始运行的时候，kubernenes会初始化一系列的环境变量指向现在存在的服务。如果你创建的服务早于客户端的pod的创建，pod上的进程可以根据环境变量获得服务的ip地址和服务号

    pod内部执行kubectl env 查看环境变量即可看到

    ![](\images\service\service-env.png)

    **Notes：** 服务名称中的横杆会被转换成下划线，并且当服务名称用作环境变量 名称前缀使，所有字母都是大写的。

2. 通过DNS发现服务

    在 kube-system 的命名空间下有个 kube-dns 的 pod和相同名字的服务，这个 pod 运行 DNS 服务，在集群中的其他pod都被配置成使用其作为dns（kubenetes通过修改每个容器的/etc/resolv.conf文件实现）。运行在pod上的进程 DNS 查询就会被 Kubernates 自身的 DNS 服务器响应，该服务知道系统中运行的所有服务。 

    **Notes：** pod也可以使用内部的dns，取决于pod的spec 的 dnsPolicy属性。 每个服务从内部 DNS 服务器中获得一个 DNS 条目，客户端的pod在知道服务名称的情况下可以通过全限定域名来访问，而不是环境变量。

3. 通过全限定域名(FQDN)连接服务

   eg：前端pod通过打开以下FQDN的连接来访问后端数据库服务: backend-database.default.svc.cluster.local  

        -backend-database: 服务名称

        -default: 表示服务所在的命名空间

        -svc.cluster.local: 所在集群本地服务名称中使用的可配置集群域后缀 

    通过cat /etc/resolv.conf 查看pod容器DNS解析器配置

### **连接集群外部的服务**

通过 Kubernates 服务特性暴露外部服务的情况。让服务将连接重定向到外部的 IP 和端口。

#### Endpoint

服务和 pod 不是直接相连的，有一种资源介于两者之间，就是 Endpoint 资源。**Endpoint 资源就是暴露一个服务的ip地址和端口的列表**。Endpoint资源和其他 kubernetes 资源一样，所以可以使用kubectl info来获取他的基本信息。

![](\images\service\end-point.png)

实际上这里的 pod 选择器，在重定向传入连接时不会直接使用它。相反，选择器用于构建 IP 和端口列表，然后存储在 Endpoint 资源中。当客户端连接到服务时，服务代理选择这些 IP 和端口对中的一个，并将传入连接重定向到在该位置监听的服务器。

#### 手动配置 Endpoint 的方式来创建服务， 连接外部服务

1. 创建没有选择器的服务

    ```
    apiVersion: v1
	kind: Service
	metadata:
	  name: external-service
	spec:
	  ports:
      - port: 8081 ## 接受端口8081上的传入连接
    ```

2. 为没有选择器的服务创建Endpoint资源

    ```
    apiVersion: v1
	kind: Endpoints
	metadata:
	  name: external-service ##和service的名字一样
	subsets:
	  - addresses:
	    - ip: 9.30.146.37 ##外部服务的地址
	    ports:
	    - port: 8081  ##外部服务的端口

    ```

    endpoint对象需要与服务具有相同的名称，并包含该服务的目标ip地址和端口列表。在服务创建后创建的容器将包含服务的环境变量，并且与其ip:port对的所有连接都将在服务端口之间进行负载均衡。

### **将服务暴露给外部客户端**

有三种方式可以在外部访问服务：

1. 将服务类型设置成 NodePort

2. 将服务的类型设置成 LoadBalance, NodePort的一种扩展

3. 创建一个 Ingress 资源，通过一个 IP 地址公开多个服务（Service 二 中介绍）

#### NodePort 暴露服务

将服务类型设置成 NodePort，每个集群节点都会在节点上打开一个端口，并将在该端口上接收到的流量重定向到基础服务。 该服务仅在内部集群 IP 和端口上才可访问，但也可通过所有节点的专有端口访问。

```
apiVersion: v1
kind: Service
metadata:
  name: kubia-nodeport
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8081
    nodePort: 30123 ## 如果不指定，Kubernates 会选择一个随机端口
  selector:
    app: kubia
```

创建好之后，可以从集群外部通过 [master IP]:30123 访问 selector 为 app:kubia 的pod

#### LoadBalance 暴露服务

将服务类型设置为LoadBalancer--NodePort类型的一种扩展--这使服务可以通过一个专用的负载均衡器来访问。负载均衡器将流量重定向到跨所有节点的节点端口。客户通过负载均衡器的ip连接到服务。如果kubernetes在不支持Load Balancer服务的环境中运行，则不会调配负载均衡器，但该服务仍表现得像一个NodePort服务。

```
apiVersion: v1
kind: Service
metadata:
  name: kubia-loadbalancer
spec:
  externalTrafficPolicy: Local ##代理服务器将选择本地运行的pod，如果本地无，则挂起，为了减少不必要的网络跳转。无法确保负载均衡器将连接转发到至少具有一个pod的节点,同时如果一个节点上有两个pod，那么可能会导致跨pod的负载分布不均衡
  type: LoadBalancer
  ports:
  - port: 80
  targetPort: 8081
  selector:
    app: kubia
```

### 笔记来源

Kubernates in Action








