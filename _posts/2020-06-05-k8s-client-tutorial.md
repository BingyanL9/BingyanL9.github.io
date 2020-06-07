---
layout: post
title:  "k8s - Java client - 教程"
date:   2020-06-05 21:18:54
categories: k8s Java-client Tutorial
tags: k8s Tutorial
excerpt: 近期工作中需要使用到 kubernates-java 客户端，发现无论是官方文档还是网上的博客都比较零碎和分散。所以决定自己整理一下教程，仅作为记录。
mathjax: true
---

* content
{:toc}

> 近期工作中需要使用到 kubernates-java 客户端，发现无论是官方文档还是网上的博客都比较零碎和分散。所以决定自己整理一下教程，仅作为记录。

> 前提: 1. 拥有 K8s 环境并且安装了 kubectl; 2. Java 开发环境

### **k8s-client-java 类型**

1. [kubernetes-client/java](https://github.com/kubernetes-client/java)

    官方支持的Java API，根据k8s-openapi随之更新，一致性和更新频率高。

2. [fabric8io/kubernetes-client](https://github.com/fabric8io/kubernetes-client)

    非官方，支持 kubernates 以及 openshift。一致性低，更新慢；其中不支持k8s1.8和1.1， OpenShift 支持到 4.2.0。

出于一致性考虑，这个教程中使用 kubernetes-client/java。

### **kubernetes-client/java 文档解释**

#### kubernetes REST API

REST API 是 Kubernates 提供给用户去操作资源的基础架构。使用 kubectl 命令行界面或者其他命令行工具去执行本身也是使用 REST API。 REST API 路径格式如下：

|Path|解释|
|----|----|
|GET/apis(api)/[api组名]/版本/[资源名的复数形式]|获取某个资源的列表|
|GET/apis(api)/[api组名]/版本/[资源名的复数形式]/[名字]|获取给定名称的单个资源|
|POST/apis(api)/[api组名]/版本/[资源名的复数形式]|创建一个资源，该资源的创建信息来自用户提供的yaml文件（最终转换成Json对象）|
|PATCH/apis(api)/[api组名]/版本/[资源名的复数形式]/[名字]|选择修改资源详细指定的域|
|PUT/apis(api)/[api组名]/版本/[资源名的复数形式]/[名字]|通过给出的资源的资源名和用户提供的yaml文件（最终转换成Json对象）来更新或者创建资源|
|DELETE/apis(api)/[api组名]/版本/[资源名的复数形式]/[名字]|通过给出的资源名删除指定的资源，删除选项（DeleteOptions）中可以指定优雅删除（Grace Deletion）的时间（GracePeriodSeconds），该配置表示从服务器接收到删除请求到删除的时间间隔，单位为秒。|
|GET/apis(api)/[api组名]/版本/watch/[资源名的复数形式]|不断接受一连串的Json对象，记录给定资源类别内的资源随时间的变化情况|

- API 组名

    - core（也称为 legacy）组，它位于 REST 路径/api/v1上，未指定为 apiVersion 字段的一部分，例如apiVersion: v1

    - 特定名称的组位于 REST 路径```/apis/$GROUP_NAME/$VERSION```下，并使用```apiVersion:$GROUP_NAME/$VERSION```（例如，apiVersion:batch/v1）。您可以在 Kubernetes API 参考 中找到受支持的 API Group 的完整列表，或者使用 ```kubectl api-versions```来获取所有支持的组名和它的版本。

    - 自定义扩展资源

- API 版本

    版本是在 API 级别而非资源或字段级别配置。因此要操作不同版本的资源需要使用不同版本的 REST，并且参考不同版本的文档。

    - 确保 API 呈现出清晰一致的系统资源和行为视图。

    - 允许控制对已寿终正寝的 API 和/或实验性 API 的访问。

- API 版本的说明

    不同版本的 API 表示不同级别的稳定性和支持级别。

    -  Alpha：

        - 版本名称包含 alpha（例如，v1alpha1）。

        - 该软件可能包含错误。启用功能可能会暴露错误。默认情况下，功能可能被禁用。

        - 对功能的支持随时可能被删除，但不另行通知。

        - 在以后的软件版本中，API 可能会以不兼容的方式更改，亦不另行通知。

        - 由于存在更高的错误风险和缺乏长期支持，建议仅在短期测试集群中使用该软件。

    - Beta：

        - 版本名称包含beta（例如，v2beta3）

        - 该软件已经过充分测试。启用功能被认为是安全的。默认情况下启用功能。

        - 尽管细节可能会发生变更，对应功能不会被废弃。

        - 在随后的 Beta 或稳定版本中，对象的模式和/或语义可能会以不兼容的方式更改。发生这种情况时，将提供迁移说明。迁移时可能需要删除、编辑和重新创建 API 对象。编辑过程可能需要一些思考。对于依赖该功能的应用程序，可能需要停机。

        - 该软件仅建议用于非关键业务用途，因为在后续版本中可能会发生不兼容的更改。如果您有多个可以独立升级的群集，则可以放宽此限制。

    - Stable

        - 版本名称为```vX```，x为正整数

        - 功能特性的稳定版本会持续出现在许多后续版本的发行软件中

    为了消除字段或重组资源表示形式，Kubernetes 支持多个 API 版本，每个版本在不同的 API 路径下。例如：/api/v1 或者 /apis/extensions/v1beta1。

在了解了REST API格式的前提下，可以打开 [Kubernates API v1.18 doc](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/)去查看：

![](\images\k8s-client-java\kubernates-rest-api.png)

左边是资源和对应的操作，右边是操作的解释和例子。

也可以使用 ```kubectl explain``` 来查看文档:

```
kubectl explain RESOUCE [option]

Options:
      --api-version='': Get different explanations for particular API version
      --recursive=false: Print the fields of fields (Currently only 1 level deep)

eg：

>> kubectl explain pod

KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.

FIELDS:
   apiVersion   <string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind <string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata     <Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec <Object>
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

   status       <Object>
     Most recently observed status of the pod. This data may not be up to date.
     Populated by the system. Read-only. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

```

#### kubernetes-client/java 文档说明

[kubernetes-client/java 文档](https://github.com/kubernetes-client/java/tree/master/kubernetes/docs)，以 api 结尾的都是我们可以调用的 API 接口。 

API 封装类的文档命名规则是：apiGroup + 版本信息 + api.md

比如要找 Ingress 的接口， 流程如下：

1. 到 [Kubernates API v1.18 doc](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/) 查看Ingress， 找到它的 Path ```/apis/networking.k8s.io/v1beta1/namespaces/{namespace}/ingresses```， 得出这个接口封装类的名字应该是 networking 开头的，并且以v1beta1Api结尾。

2. 打开[kubernetes-client/java 文档](https://github.com/kubernetes-client/java/tree/master/kubernetes/docs) 搜索networking：

    ![](\images\k8s-client-java\networkingAPi.png)

3. 找到 NetworkingV1beta1Api.md， 打开，大部分的接口都会有简单的用例。

### **配置开发环境**

1. 依赖设置

    - Maven

        将这个依赖加到 pom.xml 文件中：

        ```
        <dependency>
        <groupId>io.kubernetes</groupId>
        <artifactId>client-java</artifactId>
        <version>8.0.2</version>
        <scope>compile</scope>
        </dependency>
        ```

    - Gradle 

        ```
        compile 'io.kubernetes:client-java:8.0.2'
        ```

    - 其他 参考[kubernetes-client/java 的 Readme](https://github.com/kubernetes-client/java)

2. 获取 Kubeconfig 并复制到项目路径下

    config 文件存储着用户的登录信息和 token，位于 ```/root/.kube ```底下：

    ```
    cd /root/.kube
    ls
    ```

    ![](\images\k8s-client-java\kuberconfig.png)

    ![](\images\k8s-client-java\project_folder.png)

3. 测试是否能够了连接 kubernates

    ```
    ...

    @RunWith(SpringJUnit4ClassRunner.class)
    @ActiveProfiles("test")
    @FixMethodOrder(MethodSorters.NAME_ASCENDING)
    public class TestClient {

        private String kubeConfigPath = "kube/config";

        @Test
        public void testConnection() throws ApiException, IOException, ApiException {
            //加载k8s,confg
            ApiClient client =
                    ClientBuilder.kubeconfig(KubeConfig.loadKubeConfig(new FileReader(kubeConfigPath))).build();
    
            //将加载confi的client设置为默认的client
            Configuration.setDefaultApiClient(client);
    
            //创建一个api
            CoreV1Api api = new CoreV1Api();
            //打印所有的pod
            V1PodList list = api.listPodForAllNamespaces(null, null, null, null, null, null, null, null, null);
           assertTrue(list.getItems().size() > 0);
        }
    }
    ```

4. 可以封装自己的 Utils 了

### **配置 Service Account Authentication Token**

在一个集群中配置 kubeconfig 文件的时候， 默认会生成一个短期的，集群范围的，特定用户的token。 我们上一步骤复制出来的 config 文件中包含的就是这样的短期的，针对特定用户的token。

生产环境中的产品不断的去修改 kubernates 用户的 token 是极其不现实的， 我们需要长期的，不针对特定用户的token。

其中一种解决方案就是使用 Kubernetes service account：

1. 在 kube-system 命名空间中创建一个新的 service account

    ```
    kubectl -n kube-system create serviceaccount <service-account-name>

    expected output: serviceaccount/<service-account-name> created
    ```

    Note: 在 kube-system 命名空间中创建 service account 是比较推荐的做法(recommended good practice)，但是在别的命名空间中创建也是可以的。

2. 创建拥有集群管理权限的 clusterrolebinding 并将它绑定到第一步创建的 service account 中：

    ```
    kubectl create clusterrolebinding <binding-name> --clusterrole=cluster-admin --serviceaccount=kube-system:<service-account-name>

    expected output: clusterrolebinding.rbac.authorization.k8s.io/<binding-name> created
    ```

3. 获取 service account authentication token 的名字， 并将其赋给环境变量

    ```
    TOKENNAME=`kubectl -n kube-system get serviceaccount/<service-account-name> -o jsonpath='{.secrets[0].name}'`
    ```

4. 获取 service account authentication token 的值， 并将解码

    - MacOS, Linux, 或者 Unix 环境

        ```
        TOKEN=`kubectl -n kube-system get secret $TOKENNAME -o jsonpath='{.data.token}'| base64 --decode`
        ```

    - Windows

        ```
        1. kubectl -n kube-system get secret $TOKENNAME -o jsonpath='{.data.token}'

        2. 将第一步的出来的 token 值复制到 base64 解码器中（https://www.base64decode.org）

        3. 将解码器中解出来的值复制出来

        4. TOKEN=解码器中解出来的值
        ```

5. 将新的 service account 和它的 token 加入到 kubeconfig 中

    ```
    kubectl config set-credentials <service-account-name> --token=$TOKEN

    expected output: User "kubeconfig-sa" set.
    ```

6. 将 service account 修改为当前用户

    ```
    kubectl config set-context --current --user=<service-account-name>

    expected output: Context "context-ctdiztdhezd" modified.
    ```

7. 用新的 config 文件覆盖旧的

### **参考**

[使用 Java 操作 Kubernetes API](https://blog.csdn.net/fly910905/article/details/101345091)

[通过java 客户端 操作k8s集群](https://blog.csdn.net/lw277232240/article/details/97282278)

[Kubernetes API Overview](https://kubernetes.io/zh/docs/reference/using-api/api-overview/)

[Adding a Service Account Authentication Token to a Kubeconfig File](https://docs.cloud.oracle.com/en-us/iaas/Content/ContEng/Tasks/contengaddingserviceaccttoken.htm)







