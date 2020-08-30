---
layout: post
title:  "K8s - Service(二)"
date:   2020-08-29 21:18:54
categories: K8s Ingress readinessProbe
tags: K8s Ingress ReadinessProbe Foundation
excerpt: Ingress - 外部访问Kubernate服务的一种方式
mathjax: true
---

* content
{:toc}

> Ingress: 进入或者进入的行为；进入的权利；进入的手段或地点；入口。通过一个ip地址公开多个服务。

### **为什么需要ingress**

1. 每个LoadBalancer服务都需要自己的负载均衡器，以及独有的公有ip地址，而ingress只需要一个公网ip就能为许多服务提供访问。当客户端ingress发出http请求时，ingress会根据请求的主机名和路径决定请求转发到的服务。

2. ingress 在网络栈的应用层操作（http），并且可以提供一些服务不能实现的功能，诸如基于cookie的会话亲和性（session affinity）。

3. 必须强调的是，如果要使用ingress资源，ingress控制器是必不可少的。 

	```
	kubectl get po --all-namespaces| grep ingress 在所有命名空间范围内查看是否有ingress-controller
	看到*-ingress-controller-*说明这个集群中正在运行ingress控制器
  ```

    ![](\images\service\get-ingress-controller.png)

    如果是使用openshift环境, 那么：

    ![](\images\service\openshift-ingress-operator.png)

    会是operator而不是controller。[Operator VS Controller](https://octetz.com/docs/2019/2019-10-13-controllers-and-operators/)

### **使用Ingress**

#### **创建 Ingress**

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kubia
spec:
  rules:
  - host: kubia.example.echo-practice.apps.listed.os.fyre.ibm.com ## Ingress将域名 kubia.example.echo-practice.apps.listed.os.fyre.ibm.com 映射到你的服务
    http:
      paths:
      - path: /
        backend:
          serviceName: kube-svc
          servicePort: 80

```

#### **通过 Ingress 访问服务**

要通过 http://kubia.example.echo-practice.apps.listed.os.fyre.ibm.com 访问服务，需要确保域名解析为 Ingress控制器的IP。

```
获取 Ingress 的 IP 地址
kubectl get ingresses

# IP 在 ADDRESS 列中显示出来。
```

**确保在Ingress中配置的 host 指向 Ingress 的IP地址：**

将以上的 IP 地址，通过配置 DNS 服务器将 kubia.example.echo-practice.apps.listed.os.fyre.ibm.com 解析为该地址， 或者 在 ```/etc/hosts``` 文件（Windows 是 C:\windows\system\32\drivers\etc\hosts） 中添加上域名和地址的匹配

#### **了解 Ingress 的工作原理**

1. 客户端首先对 kubia.example.echo-practice.apps.listed.os.fyre.ibm.com 执行 DNS 查找， DNS 服务器（或者本地操作系统）返回了 Ingress 控制器的 IP

2. 客户端容然后向 Ingress 控制器 发送 HTTP 请求， 并在 HOST 头中指定 kubia.example.echo-practice.apps.listed.os.fyre.ibm.com。

3. 控制器从该头部确定客户端尝试访问哪个服务， 通过与服务相关联的 EndPoint 对象查看 pod IP， 并将客户端请求转发给其中一个 pod。

**Ingress 控制器不会将请求转发给该服务，只是用它选择一个pod。** 大部分控制器都是这样工作的。

#### **通过相同的 Ingress 暴露多个服务**

1. 将不同的服务映射到相同主机的不同路径

    ```
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: kubia
    spec:
      rules:
      - host: kubia.example.echo-practice.apps.listed.os.fyre.ibm.com ## Ingress将域名 kubia.example.echo-practice.apps.listed.os.fyre.ibm.com 映射到你的服务
        http:
          paths:
          - path: /srv1
            backend:
              serviceName: kube-svc1
              servicePort: 80
          - path: /srv2
            backend:
              serviceName: kube-svc2
              servicePort: 80
    ```

2. 将不同的服务映射到不同主机上

    ```
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: kubia
    spec:
      rules:
      - host: kubia.example1.echo-practice.apps.listed.os.fyre.ibm.com ## Ingress将域名 kubia.example.echo-practice.apps.listed.os.fyre.ibm.com 映射到你的服务
        http:
          paths:
          - path: /
            backend:
              serviceName: kube-svc
              servicePort: 80
      - host: kubia.example2.echo-practice.apps.listed.os.fyre.ibm.com ## Ingress将域名 kubia.example.echo-practice.apps.listed.os.fyre.ibm.com 映射到你的服务
        http:
          paths:
          - path: /
            backend:
              serviceName: kube-svc1
              servicePort: 80

    ```

    DNS 需要将上面的两个  HOST 域名都指向 Ingress 控制器的 IP 地址。

#### **配置 Ingress 处理 TLS 运输**

为了能使 Ingress 控制器转发 HTTPS 请求， 我们需要对 Ingress 进行配置以支持 TLS。

当客户端创建到 Ingress 控制器的 TLS 连接时，客户端到控制器之间的通信是加密的，而控制器到后端 pod 之前的通信不是。 运行在 pod 上的应用程序不需要支持 TLS。 

1. 为 Ingress 创建 TLS 认证， 将证书和私钥附加到 Ingress。 这两个必需的资源存储在成为 Secret 的 K8s 的资源中， 然后 Ingress manifest 中引用它。

    ```
    openssl genrsa -out tls.key 2048
    openssl req -new -x509 -key tls.key -out tls.cret -days 360 -subj /CN=kubia.example.echo-practice.apps.listed.os.fyre.ibm.com
    kubectl create secret tls tls-secret --cert=tls.cret --key=tls.key
    secret/tls-secret created
    ```

2. 更新 Ingress 对象：

    ```
    apiVersion: extensions/v1beta1
    kind: Ingress
    metadata:
      name: kubia
    spec:
      tls:
        - hosts:
            - kubia.example.echo-practice.apps.listed.os.fyre.ibm.com
          secretName: tls-secret
      rules:
        - host: kubia.example.echo-practice.apps.listed.os.fyre.ibm.com
          http:
            paths:
              - path: /
                backend:
                  serviceName: kube-svc
                  servicePort: 80
    ```

现在可以使用 HTTPS 通过 Ingress 访问服务了：

```
curl -k -v https://kubia.example.echo-practice.apps.listed.os.fyre.ibm.com/
```

![](\images\service\visitThroghHTTPS.png)

### **pod 就绪后发出信号**

只要创建了具有适当标签的新 pod， 它就成为服务的一部分， 并且请求开始被重定向到pod。 但是如果 pod 没有准备好呢？

#### **就绪探针**

就绪探测器会定期调用，并确定特定的 pod 是否接收客户端的请求。 当容器的准备就绪探测返回成功时，表示容器已准备好接受请求。

就绪指针的类型：

1. Exec 探针，执行进程的地方。 容器的状态由进程的退出状态代码确定。

2. HTTP GET 探针，向容器发送 HTTP GET 请求，通过响应的 HTTP 状态码判断容器是否准备好。

3. TCP socket 探针，他打开一个 TCP 连接到容器的制定端口。 如果连接已经建立，则认为容器准备就绪。

#### **就绪探针的操作**

启动容器时，可以为 K8s 配置一个**等待时间**，经过等待时间后才可以执行第一次准备就绪检查。之后，他会周期性地调用探针，并根据就绪探针的结构采取行动。

如果某个 pod 报告它尚未准备就绪，就会从该服务对应的 Endpoint 中删除该 pod。 如果再次准备就绪，则重新添加 pod。

#### **为什么就绪指针重要**

假设一组 pod 依赖另外一组 pod 提供的服务。如果任何一个前端连接点出现连接问题并且无法再访问数据库，那么就绪探针就会告诉 K8s 该 pod 没有准备好处理任何请求。 

如果其他的 pod 实例没有遇到类似的连接问题，则它们可以正常处理请求。**就绪探针确保客户端与正常的 pod 交互，并且不会知道系统存在问题**。

#### **向 pod 中添加就绪探针**

1. 通过 ```kubectl edit pod kubia``` 向已存在的 pod 添加探针模板

2. 修改 yaml 文件，  ```kubectl apply -f ×××.yaml*``` 

    ```
    ##http 
    apiVersion: v1
    kind: Pod
    metadata:
      name: kubia
      labels:
        apps: kubia
    spec:
      containers:
      - image: echoibm/kubernate-test:latest
        name: kubia
        ports:
        - containerPort: 8081
          protocol: TCP
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          timeoutSeconds: 5
    ##exec

    apiVersion: v1
    kind: Pod
    metadata:
      name: kubia
      labels:
        apps: kubia
    spec:
      containers:
      - image: echoibm/kubernate-test:latest
        name: kubia
        ports:
        - containerPort: 8081
          protocol: TCP
        readinessProbe:
          exec:
            command:
            - ls
            - /var/ready
          initialDelaySeconds: 30
          timeoutSeconds: 5
    ```

如果没有将就绪探针添加到 pod 中，他们几乎是立即成为服务端点。如果应用启动但尚未准备好接受传入连接时，客户端请求将被转发到该 pod。 因此，客户端会看到连接被拒绝类型的错误。

**所以应该始终定义一个就绪探针**。

### **排除服务故障**

如果无法通过服务访问 pod， 应该根据以下列表进行排查：

1. 确保从集群内部连接到集群的ip，而不是外部

2. 不要通过 ping 服务 IP 来判断服务是否可访问（服务的 ip 是虚拟 ip ，无法ping通）

3. 如果定义就绪探针，请确保它返回成功；否则该 pod 不会成为服务的一部分。

4. 要确认某个容器是服务的一部分， 使用一下命令来检查相应的端点对象

     ```

     kubectl get endpoints

     ```

5. 如果尝试通过 FQDN 或其中的一部分来访问服务：

    ```
    eg:

    myservice.mynamespace.svc.cluster.local
    myservice.mynamespace
    ```

不起作用，请检查是否可以使用其集群 IP 而不是 FQDN 来访问。

6. 检查是否连接到服务公开的端口，而不是目标端口

7. 尝试直接连接到 pod IP 以确认 pod 正接受正确端口上的连接

8. 如果甚至无法通过 pod IP 访问应用， 请确保应用不是仅绑定在本地主机上

### 笔记来源

Kubernates in Action

延伸阅读：
[Operator VS Controller](https://octetz.com/docs/2019/2019-10-13-controllers-and-operators/)









