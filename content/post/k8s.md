---
title: "k8s介绍"
date: 2020-06-09T17:21:34+08:00
draft: false
---

## 一、Kubernetes 发展历史
#### 1. 资源管理器框架：`MESOS` -> `Docker Swarm` -> `Kubernetes`
1. **`MESOS`** : `APACHE下的分布式资源管理框架`，2019-5 Twitter 抛弃 MESOS 转向 Kubernetes
2. **`Docker Swarm`** : `Docker的集群管理工具`，2019-07 阿里云从列表中剔除 Docker Swarm
3. **`Kubernetes`** : `开源容器集群管理系统`，由Google公司开发，它的前身是 Borg 系统。用于自动部署、资源调度、服务发现、扩容缩容等功能。
- 特点：
  - 轻量级：采用 go 语言，消耗资源小
  - 开源，有可扩展性和容错性
  - 硬件资源利用高
  - 弹性伸缩
  - 负载均衡
  - 滚动升级

## 二、基础概念： 

#### 1. k8s 几种部署安装方式
- **Minikube** : 单机版k8s集群，开发测试用户使用。不能用于生产环境。
- **Kubeadm** : `kubeadm init`和`kubeadm join`命令，快速部署k8s集群。
- **二进制包部署** : 手动部署每个组件，组成k8s集群。

Kubeadm 和 二进制包部署区别：
- 生产环境中主要使用Kubeadm或二进制包部署
- Kubeadm降低部署门槛，屏蔽了很多细节，不了解细节，遇到问题很难排查。
- 二进制包部署，部署麻烦，但学习很多工作原理，有利于后期维护。


#### 2. 什么是 Pod
- Pod 是 k8s 中创建的最小部署单元
- 一个 Pod 可包含一个或者多个容器，多个容器之间共享存储、网络栈等。  

Pod 分两类：
- **自主式 Pod** : 被删除或死亡后将不再生存。
- **控制器管理的 Pod** :  在集群范围内提供`自动伸缩、自动修复`能力。

#### 3. Kubernets 对象有：
- Pod
- Service
- Volume （Storage对象）
- Namespace （Metadata对象）

高级抽象对象：
- Deployment
- ReplicaSet
- StatefulSet
- DaemonSet
- HPA
- Job
- CronJob

k8s对象 描述文件的字段（4个大字段）：
- apiVersion： 调用 k8s API 接口 使用的版本；
- kind：创建的对象类型，例如 Pod，Deployment 等；
- metadata：区分对象的元数据，包括：name，UID 和 namespace 等；
- spec： 对象的其它描述信息。不同类型对象有不同的spec定义。


#### 4. 组件说明：
##### **1. Master Node 组件** ：控制管理组件
- **`Kubectl`** ： 是k8s集群的`CLI命令行管理工具`，提供`创建和管理k8s组件`的各种方法。
- **`kube-apiserver`** : k8s API 服务组件，所有服务访问统一入口。
- **`etcd`** : 一致性、高可用性的`分布式键值数据库`，保存 K8s 集群`所有配置数据`，负责节点间的`配置共享、服务发现`。
- **`kube-scheduler`** : `Pod调度器`，根据`资源需求`选择合适的Node进行分配。
- **`kube-controller-manager`** : 集群的控制管理中心
  - **`Node Controller`**:`节点控制器`，负责在节点出现故障时，及时通知响处理。
  - **`Replication Controller & ReplicaSet`**: `副本控制器`，维护Pod副本期望数目。ReplicaSet 支持集合式的selector标签选择。
  - **`Endpoints Controller`** : `端点控制器`，生成和维护Endpoints对象，监听Service和对应的Pod的变化。
  - **`Service Account & Token Controllers`** : `服务帐户和令牌控制器`，用来创建默认帐户和 API 访问令牌.
- **`cloud-controller-manager`** ：云控制器管理器
  - **`Node Controller`** : `节点控制器`，检查云提供商节点，停止响应后立即删除
  - **`Route Controller`** : `路由控制器`，云基础架构中设置路由
  - **`Service Controller`** : `服务控制器`，创建、更新、删除云提供商负载均衡器
  - **`Volume Controller`** : `数据卷控制器`，创建、附加、装载云提供商提供卷

##### **2. Minion Node 组件** ：提供 K8s 节点运行环境，维护Pod的运行。(Minions:下属; 小卒; 杂役)
- **`kube-proxy`** `网络代理组件`: 维护Node上的网络规则，监控Pod的变化并创建相应的 ipvs rules。
- **`kubelet`** : `节点代理组件`，功能：管理pod 生命周期、容器健康检查、容器监控。

##### 3. 其它组件
- **`CoreDNS`** ：每个Service对象都会创建DNS记录，为`集群的Pod`提供`DNS解析服务`
- **`Dashboard`** ：K8s Web 控制台，可视化管理集群中`运行的应用`，方便查看`故障排除`
- **`Ingress`** ：七层网络代理
  - Ingress Controller：负责实现路由规则。
  - Ingress：负责描述域名路由规则。
- **`Federation`**（联邦）：提供跨集群中心多K8S的统一管理功能
- **`Prometheus`**：提供K8S集群的监控能力
- **`ELK`**：提供 K8S 集群日志收集，统一分析平台

## 三、Kubernetes 安装构建集群
#### 1. 系统初始化配置
- 设置 Node hostname
- 设置防火墙为 Iptables
- 关闭 SELINUX
- 调整内核参数
- 调整系统时区时间
- 关闭系统不需要服务
- 设置 rsyslogd 和 systemd journald
- 升级系统内核为 4.44

#### 2. Kubeadm 安装部署
- 开启 ipvs 模块
- 安装 Docker 
- 安装 Kubeadm
- 初始化 Master 节点
- 部署 Flannel 扁平化网络
- 添加 Worker Node

## 四、资源
#### 1. 资源概念：K8s中所有的内容都抽象为资源，资源实例化后叫做对象
##### 资源分类
- 工作负载型资源(workload) 7个：Pod、Deployment、ReplicaSet、StatefulSet、DaemonSet、Job、CronJob
- 服务发现及负载均衡型资源(Service Discovery LoadBalance):Service、Ingress 等
- 配置与存储型资源：Volume(存储卷)、ConfigMap(配置中心)、Secret(保存敏感数据)、DownwardAPI(外部配置资源引入)
- 集群级别的资源：Namespace、Node、Role、ClusterRole、RoleBinding、ClusterRoleBinding 
- 元数据型资源(通过指标操作的)：HPA、PodTemplate、LimitRange

#### 2. Pod 的生命周期
- **`Pod Pause容器`** : 初始化`网络和数据卷`
- **`initContainers`** : 初始化容器，在容器启动的时候，会先启动可一个或多个initC容器，如果有多个，那么这几个InitC按照定义的顺序依次执行，只有所有的InitC执行完后，主容器才会启动。
  - 可做一些初始化的配置
  - 可以等待其它模块Ready
- **容器探针**：由 kubelet 定期执行对容器诊断
  - **`readinessProbe`** (就绪探测): `探测容器是否准备好`。 如果探测失败，`端点控制器`将从匹配的所有Service的端点中删除该Pod的IP。
  - **`livenessProbe`** (存活探测): `探测容器是否在运行`。如果探测失败，则kubelet会杀死容器，容器受到其重启策略的影响。
  - 健康状态诊断方式（3种）：
    - `ExecAction` : 执行一个命令，返回状态为0表示成功
    - `TCPSocketAction` : 通过TCP端口尝试建立连接
    - `HTTPGetAction` : 发送HTTP GET请求
- **容器生命周期 `lifecycle` 下的 `Hook` 挂接处理函数:**
  - **`postStart`**: 容器创建完成之后，k8s 将立即发送 postStart 钩子事件
  - **`preStop`**:  在容器被终止之前， k8s 将发送一个 preStop 钩子事件
- **重启策略** : PodSpec的restartPolicy字段
  - **`Always`** ： 默认策略，Pod对象终止就重启
  - **`OnFailure`**  ： 仅在Pod对象出现错误时才重启
  - **`Never`** ： 从不重启
- **`Pod phase`**: Pod 的运行阶段（phase） ： 5个
  - **`Pending`**（挂起）：Pod容器镜像正在创建。等待时间包括：调度 Pod 的时间和通过网络下载镜像的时间。
  - **`Running`**（运行中）：该 Pod 已经绑定到了一个节点上，Pod 中所有的容器都已被创建。至少有一个容器正在运行，或者正处于启动或重启状态。
  -  **`Succeeded`**（成功）：Pod 中的所有容器都被成功终止，并且不会再重启。
  - **`Failed`**（失败）：Pod 中的所有容器都已终止了，并且至少有一个容器是因为失败终止。也就是说，容器以非0状态退出或者被系统终止。
  - **`Unknown`**（未知）：因为某些原因无法取得 Pod 的状态，通常是因为与 Pod 所在主机通信失败。

## 五、控制器：

#### 1. 什么是控制器
Kubernetes 中内建了很多 controller（控制器），相当于一个状态机，用来控制 Pod 的具体状态和行为

#### 2. 控制器类型：8个
- **`ReplicationController 和 ReplicaSet` 副本控制器**
  - 维护Pod副本期望数目。ReplicaSet 支持集合式的selector标签选择。
- **Deployment**
  - Deployment 为 ReplicaSet 和 Pod 提供了一个声明式定义 (declarative) 方法。
  - 典型的应用场景包括：
    - 创建 ReplicaSet 和 Pod
    - 滚动升级、回滚应用
    - 扩容、缩容
    - 暂停、继续
- **DaemonSet**
  - 确保每个 Node 上运行一个 Pod 的副本。当有 Node 加入集群时，也会为他们新增一个
  - 使用 DaemonSet 的一些典型用法，在每个 Node 上运行：
    - 运行集群存储 daemon，例如：`mfs`、`glusterd` 、 `ceph`
    - 运行日志收集 daemon，例如： `fluentd` 、`Filebeat`、 `logstash`、
    - 运行监控 daemon，例如 [Prometheus Node Exporter](https://github.com/prometheus/node_exporter)、`collectd`、Datadog 代理、New Relic 代理，或 Ganglia gmond
- **StatefulSet** : 有状态服务
  - StatefulSet Controller 为 Pod **提供唯一的标识**。它可以**保证部署和 scale 的顺序**。
  - 其应用 场景包括：
    - **稳定的持久化存储**，即Pod重新调度后还是能访问到相同的持久化数据，基于PVC来实现
    - **稳定的网络标志**，  即Pod重新调度后其PodName和HostName不变，基于Headless Service（即没有Cluster IP的Service）来实现
    - **有序部署，有序扩展**，即Pod是有顺序的，在部署或者扩展的时候要依据定义的顺序依次依次进行（即从0到N-1，在下一个Pod运行之前所有之前的Pod必须都是Running和Ready状态），基于init containers来实现
    - **有序收缩，有序删除**（即从N-1到0）
- **Job**
  - Job 负责批处理任务，即仅执行一次的任务，它保证批处理任务的一个或多个 Pod 成功结束
- **CronJob**  管理基于时间的 Job，即： 
  - 在给定时间点调度 Job 运行
  - 创建周期性运行的 Job，例如：数据库备份、发送邮件
- **Horizontal Pod Autoscaling** ： 控制 Pod 水平自动缩放
  - 应用的资源使用率通常都有高峰和低谷的时候，使用 HAP 可以削峰填谷，让service中的 Pod 个数自动调整，提高集群的整体资源利用率。


## 六、Service 服务发现
- **Service 说明**
  - 通过 Label Selector 方式匹配一组 Pod，使其能够通过 Service 对外提供统一访问，通常称为微服务。
  - Service 提供 4 层负载均衡能力 `round-robin` 轮询算法（基于ip和端口转发），没有 7 层功能。
- **服务发现原理**
  - **endpoint** : 是k8s集群中的一个资源对象，存储在etcd中，用来记录一个service对应的所有pod的访问地址和端口。
  - 当service配置selector时，endpoint controller才会自动创建对应的endpoint对象；否则，不会生成endpoint对象.
  - **Endpoints Controller** ： 是k8s集群控制器的其中一个组件，功能：
    - 负责生成和维护所有endpoint对象的控制器
    - 负责监听service和pod的变化，做对应更新
- **Service 常见分类**
  - **ClusterIP** ： 默认类型，自动分配一个仅 Cluster 内部可以访问的虚拟 IP
  - **NodePort**  : 在 ClusterIP 基础上，在每个Node上打开一个相同的端口以供外部访问，通过 NodePort 来访问该服务
  - **Headless Service** : 无头服务
    - 指定 Cluster IP(spec.clusterIP) 的值为 “None”，不会被分配ClusterIP。
    - 使用场景
      - 客户端负载：Headless Services不会分配ClusterIP,而是将Endpoints（pod IP数组）返回，也就将服务端的所有节点地址返回，让客户端自行要通过负载策略完成负载均衡
      - 访问域名格式：**`svc名称.命名空间名称.集群域名`**  eg: **`nginx-demo.default.svc.cluster.local`**
      - 通常配合 StatefulSet 使用。
  - **LoadBalancer**：在 NodePort 的基础上，借助 cloud provider 创建一个外部负载均衡器，并将请求转发到: NodePort
  - **ExternalName** : 把集群外部的服务引入到集群内部来（定义一个 svc对接外部地址信息），在集群内部直接使用 svc。
- **Service 实现方式**
  - userspace ： k8s v1.0 ，每次访问 pod 都需要经过 kube-proxy ，压力大。
  - iptables ： k8s v1.1 ，访问 pod 直接通过 iptables 代理完成。
  - ipvs :  在k8s 1.14 版本开始默认使用 ipvs 代理。
- **Ingress**  七层协议代理
  - Ingress Controller可以配置为提供服务外部访问的URL，负载均衡，SSL，提供基于名称的虚拟主机等。

## 七、Volume 存储：
掌握多种存储类型的特点  
并且能够在不同环境中选择合适的存储方案（有自己的简介）

- **configMap** 
  - ConfigMap是用来存储配置文件的k8s资源对象，所有的配置内容都存储在etcd中。可以用作环境变量、命令行参数或者存储卷中的配置文件。
  - ConfigMap 将您的环境配置信息和 容器镜像 解耦，便于应用配置的修改。当您需要储存机密信息时可以使用 Secret 对象。
  - configMap 热更新
    - 使用该 ConfigMap 挂载的 Env 环境变量不会同步更新
    - 使用该 ConfigMap 挂载的 Volume 中的数据需要一段时间（实测大概10秒）才能同步更新
- **Secret** 
  - Secret 解决了密码、token、密钥等敏感数据的配置问题，而不需要把这些敏感数据暴露到镜像或者 Pod Spec中。Secret 可以以 Volume 或者环境变量的方式使用
  - **Secret分类**
    - **Service Account** ： 用来访问 Kubernetes API，由 Kubernetes 自动创建，并且会自动挂载到 Pod 的/run/secrets/kubernetes.io/serviceaccount目录中
    - **Opaque Secret** ： base64编码格式的Secret，用来存储密码、密钥等
    - **kubernetes.io/dockerconfigjson**  ： 用来存储私有 docker registry 的认证信息
- **Volume**
  - 容器之中的数据都是临时的。如果容器重启，这些数据会丢失。Volume用于容器保存需要持久化的数据。多个容器之间可以共享这些数据。
  - 卷的类型：
    - `awsElasticBlockStore` `azureDisk` `azureFile` `cephfs` `csi` `downwardAPI` **`emptyDir`**
    - `fc` `flocker` `gcePersistentDisk` `gitRepo` `glusterfs` **`hostPath`** `iscsi` `local` `nfs`
    - `persistentVolumeClaim` `projected` `portworxVolume` `quobyte` `rbd` `scaleIO` `secret`
    - `storageos` `vsphereVolume` 
  - emptyDir 卷
    - 当 Pod 被分配给节点时，首先创建emptyDir卷，并且只要该 Pod 在该节点上运行，该卷就会存在。当 Pod 从节点中删除时，emptyDir中的数据将被永久删除。
    - 用途假设
      - 暂存空间，例如用于基于磁盘的合并排序
      - 用作长时间计算崩溃恢复时的检查点
      - Web服务器容器提供数据时，保存内容管理器容器提取的文件
  - hostPath 卷
    - 将主机节点的文件系统中的文件或目录挂载到集群中
      - 运行需要访问 Docker 内部的容器；使用 /var/lib/docker 的 hostPath
      - 在容器中运行 cAdvisor；使用 /dev/cgroups 的 hostPath
      - 可以把三方存储集群共享到主机，然后使用 hostPath 挂载到容器
- **PV - PVC**
  - **PersistentVolume（PV）**
      - PV 是由管理员设置的存储，也是集群中的资源。 PV 是Volume 之类的卷插件，但具有独立于使用 PV 的 Pod 的生命周期。 
    - PV 访问模式说明
      - ReadWriteOnce（RWO） —— 该卷可以被单个节点以读/写模式挂载
      - ReadOnlyMany （ROX） —— 该卷可以被多个节点以只读模式挂载
      - ReadWriteMany（RWX） —— 该卷可以被多个节点以读/写模式挂载
    - 回收策略
      - Retain（保留）——手动回收
      - Recycle（回收）——基本擦除（rm -rf /thevolume/*） （废弃）
      - Delete（删除）——关联的存储资产（例如 AWS EBS、GCE PD、Azure Disk 和 OpenStack       - Cinder 卷）将被删除
      - 当前，只有 NFS 和 HostPath 支持回收策略。AWS EBS、GCE PD、Azure Disk 和 Cinder 卷支持删除策略
    - 状态
      - Available（可用）——一块空闲资源还没有被任何声明绑定
      - Bound（已绑定）——卷已经被声明绑定
      - Released（已释放）——声明被删除，但是资源还未被集群重新声明
      - Failed（失败）——该卷的自动回收失败 
  - **PersistentVolumeClaim（PVC）**
    - 是用户存储的请求。PVC 消耗 PV 资源。 PVC 跟PV 绑定是一对一的映射。
    - 当 pod 删除后 PVC 依然会存在系统中和 pv 有绑定关系。


## 八、Scheduler 调度器：
- 调度器概念
  - Scheduler 调度器，把定义的 pod 合理的分配到集群的节点上。
  - 调度过程：分为几个部分
    - **1. `predicate` 预选**：首先是过滤掉不满足条件的节点；
      - PodFitsResources：节点上剩余的资源是否大于 pod 请求的资源
      - PodFitsHost：如果 pod 指定了 NodeName，检查节点名称是否和 NodeName 匹配
      - PodFitsHostPorts：节点上已经使用的 port 是否和 pod 申请的 port 冲突
      - PodSelectorMatches：过滤掉和 pod 指定的 label 不匹配的节点
      - NoDiskConflict：已经 mount 的 volume 和 pod 指定的 volume 不冲突
    - **2. `priority` 优选**：然后对通过的节点按照优先级排序；
      - LeastRequestedPriority：通过计算 CPU 和 Memory 的使用率来决定权重，使用率越低权重越高。这个优先级指标倾向于资源使用比例更低的节点
      - BalancedResourceAllocation：节点上 CPU 和 Memory 使用率越接近，权重越高。这个应该和上面的一起使用。
      - ImageLocalityPriority：倾向于已经有要使用镜像的节点，镜像总大小值越大，权重越高 
    - **3. 最后从中选择优先级最高的节点。**
  - 自定义调度器
    - 通过spec:schedulername参数指定调度器的名字，可以为 pod 选择某个调度器进行调度。
- 调度亲和性 （为了更细粒度的去控制 Pod 的调度）
  - nodeAffinity，nodeAntiAffinity ： 节点，亲和性  和反亲和性
    - preferredDuringSchedulingIgnoredDuringExecution ： 软策略
    - requiredDuringSchedulingIgnoredDuringExecution ： 硬策略
  - podAffinity，podAntiAffinity ： Pod 亲和性 和 反亲和性
    - preferredDuringSchedulingIgnoredDuringExecution ： 软策略
    - requiredDuringSchedulingIgnoredDuringExecution ： 硬策略
  - 亲和性运算符
    - **In**：label 的值在某个列表中
    - **NotIn**：label 的值不在某个列表中
    - **Gt**：label 的值大于某个值
    - **Lt**：label 的值小于某个值
    - **Exists**：某个 label 存在
    - **DoesNotExist**：某个 label 不存在
- 污点
  - `taint` 命令可以给某个 Node 节点设置污点，Node 被设置上污点之后就和 Pod 之间存在了一种相斥的关系，可以让 Node 拒绝 Pod 的调度执行，甚至将 Node 已经存在的 Pod 驱逐出去
  - Taint
    - 污点的组成 : `key=value:effect`
    - 删除污点只需要在最后增加一个-： `key=value:effect-`
    - effect 支持如下三个选项：
      - NoSchedule：表示 k8s 将不会将 Pod 调度到具有该污点的 Node 上
      - PreferNoSchedule：表示 k8s 将尽量避免将 Pod 调度到具有该污点的 Node 上
      - NoExecute：表示 k8s 不会将 Pod 调度到具有该污点的 Node 上，同时会将 Node 上已经存在的 Pod 驱逐出去
  - Tolerations
    - 可以在 Pod 上设置容忍 (Toleration) 污点的存在，设置后 pod 可以被调度到存在污点的此 Node 上。
    - 选项：
      - 其中 key, vaule, effect 要与 Node 上设置的 taint 保持一致
      - operator 的值为 Exists 将会忽略 value 值
      - tolerationSeconds 用于描述当 Pod 需要被驱逐时可以在 Pod 上继续保留运行的时间
- 固定节点调度
  - `Pod.spec.nodeName` 将 Pod 直接调度到指定的 Node 节点上，跳过 Scheduler 的调度策略
  - `Pod.spec.nodeSelector`node节点标签选择器调度：标签：key=value

## 九、集群安全机制：
集群的**认证，鉴权，访问控制**，原理及其流程 

- **认证**
  - HTTP Token ： 通过一个 Token 来识别合法用户
  - HTTP Base ： 通过用户名+密码的方式认证
  - HTTPS ：基于 CA 根证书签名的客户端身份认证方式
- **鉴权**
  - AlwaysDeny ： 表示拒绝所有的请求，一般用于测试
  - AlwaysAllow：允许接收所有请求，如果集群不需要授权流程，则可以采用该策略
  - ABAC（Attribute-Based Access Control） ：基于属性的访问控制，表示使用用户配置的授权规则对用户请求进行匹配和控制
  - Webbook ：通过调用外部 REST 服务对用户进行授权
  - RBAC ：（Role-Based Access Control）：基于角色的访问控制，现行默认规则
    - Role and ClusterRole
      - Role 可以定义在一个 namespace 中，如果想要跨 namespace 则可以创建ClusterRole 
      - ClusterRole 具有与 Role 相同的权限角色控制能力，不同的是 ClusterRole 是集群级别的
    - RoleBinding and ClusterRoleBinding
      - RoloBinding 可以将角色中定义的权限授予用户或用户组
      - RoleBinding 适用于某个命名空间内授权，而 ClusterRoleBinding 适用于集群范围内的授权
    - Resources
      - k8s 集群内一些资源一般以其**名称字符串**来表示
      - 如果要在 RBAC 授权模型中控制这些子资源的访问权限，可以通过 / 分隔符来实现; eg： `resources: ["pods/log"]`
    - to Subjects
      - RoleBinding 和 ClusterRoleBinding 可以将 Role 绑定到 Subjects；Subjects 可以是 groups、users 或者service accounts
- **准入控制** : 通过 Admission Control 实现
  - 是 k8s API Server 用于拦截请求的一种方式。Admission 可以做到对请求的资源对象进行校验，修改。
  - 准入控制是API Server的插件集合，通过添加不同的插件，实现额外的准入控制规则。
    - NamespaceLifecycle：防止在不存在的 namespace 上创建对象，防止删除系统预置 namespace，删除namespace 时，连带删除它的所有资源对象。
    - LimitRanger：确保请求的资源不会超过资源所在 Namespace 的 LimitRange 的限制。
    - ServiceAccount：实现了自动化添加 ServiceAccount。
    - ResourceQuota：确保请求的资源不会超过资源的 ResourceQuota 限制。
#### 最佳安全措施
- 定期安全更新：计划每季度至少升级一次
- 使用命名空间：不同类型的工作负载，部署在不同的命名空间，实现安全隔离
- 限制对etcd的访问
- 记录生产环境中的所有内容
- 实施网络策略细分
- 制定严格的资源策略/规则
- 实施持续安全漏洞扫描
- 启用审核日志
- 限制对k8s节点的直接访问
- 定义资源配额
- 仅使用已认证的仓库镜像

## 十、Helm
Helm 让 K8s 的应用管理（Deployment,Service 等 ) 可配置，能动态生成。通过动态生成 K8s 资源清单文件（deployment.yaml，service.yaml）。然后调用 Kubectl 自动执行 K8s 资源部署Helm

- Helm 有两个重要的概念：**chart** 和 **release**
  - **chart** 是创建一个应用的信息集合，包括各种 k8s 对象的配置模板、参数定义、依赖关系、文档说明等。chart 是应用部署的自包含逻辑单元。
  - **release** 是 chart 的运行实例，代表了一个正在运行的应用。当 chart 被安装到 Kubernetes 集群，就生成一个 release。chart 能够多次安装到同一个集群，每次安装都是一个 release
- Helm 目录文件结构
  ```
  ├── charts            # 用于放置子charts
  ├── Chart.yaml        # 这里有一些chart的基本信息，名称、版本、对应的app版本等。
  ├── templates         # 模板（资源清单） 最终被渲染为不同的 manifest
  │   ├── deployment.yaml # 这个模板最终会被渲染为 deployment的基本manifest
  │   ├── _helpers.tpl    # 这个文件用于 存放模板，给其他地方调用，本身不会被渲染（参照函数库理解）
  │   ├── ingress.yaml      
  │   ├── NOTES.txt       # chart的说明文件，会在安装、升级后显示其内容，也可以用模板的方式写，会被渲染。
  │   └── service.yaml
  └── values.yaml       # 存放环境变量‘值’的地方，渲染templates下模板所用的值
  ```
- HELM 部署实例
  - dashboard : 基于网页的 k8s 用户界面
  - metrics-server : 集群范围资源使用数据的聚合器，来采集指标信息
  - Prometheus : k8s 集群监控
  - Grafana : 时序数据的可视化分析工具
  - Heapster：是一个收集者，将每个Node上的cAdvisor的数据进行汇总，然后导到第三方工具(如InfluxDB)。
  - cAdvisor：收集本机以及容器的监控数据(cpu,memory,filesystem,network,uptime)
  - InfluxDB：开源的分布式时序数据库
  - EFK : 日志收集解决方案: Elasticsearch、Fluentd 和 Kibana

		

## 十一、运维：
- 修改Kubeadm 达到证书可用期限为 10年
- 能够构建高可用的 Kubernetes 集群，高可用集群副本数据最好 >= 3（且是奇数个）
- 服务分类
  - 有状态服务：需要数据存储功能的服务、或者指多线程类型的服务，队列等。
    - 如： Mysql、Kafka、ZooKeeper 等  
  - 无状态服务：不会在本地存储需要持久化的数据
    - 如： LVS、Apache、Nginx、Tomcat 等

#### K8S SDN网络解决方案
SDN是Software-defined networking的缩写，在k8s中需要使用某种SDN技术用以解决“每个Pod有独立IP”，这是基本的要求。

##### 容器网络技术方案选型推荐
任何的技术方案都离不开场景，在这里我们根据不同的场景给大家推荐几种技术方案：
- 单服务器：不需要网络组件，使用Docker自带的网络即可
- 小规模集群：使用kubenet + hostroutes，简单、易配合管理
- 云环境中的小规模集群：使用kubenet + master组件上运行的网络控制器，充分利用IaaS所提供的VPC环境中的路由功能，简化网络配置
- 服务器不在一个网段的集群：使用Flannel提供的vxlan或者其他类似的Tunnel技术
- 安全要求高的集群：使用Calico或者Open vSwitch等支持Policy的SDN技术




#### Federation 集群联邦
- Kubernetes 官方声称单集群最多可支持`5000个节点`和`15万个Pod`
- 使用`kubefed`部署`federation`， 使用`kubefed join` 注册集群
- 在联邦集群的帮助下，可以将多个k8s集群作为单个集群进行管理。可以创建多个k8s集群，并使用联邦功能在一个位置控制管理它们。

#### Operator
- Operator 是由 CoreOS 开发的，用来扩展 Kubernetes API 特定的应用程序控制器。
- 它用来创建、配置和管理复杂的有状态应用，如数据库、缓存和监控系统。
- Operator 基于 k8s 的资源和控制器概念之上构建，但同时又包含了应用程序特定的领域知识。
- 创建Operator 的关键是CRD（自定义资源）的设计。


#### gRPC
- gRPC 是一个现代化高性能开源远程过程调用（RPC）框架。
- CoreOS 的分布式键值存储 etcd 就使用了 gRPC 进行点对点通讯。


#### Istio 服务网格 (Service Mesh)
- Istio 是一个开源的服务网格(Service Mesh) 平台，用于为`分布式微服务架构`服务治理。
- Istio + k8s 是 Service Mesh 最流行的原生应用服务治理实现方式
- k8s 原生 service方案：可以做负载均衡，但是缺乏熔断、限流、灰度等操作
- SideCar (底层使用 Envoy) 从容器，在每个 pos 中起个 Sicecar(istio-proxy) 容器，再由 istio-proxy 做流量改写、拦截、转发，来实现熔断、限流、灰度等操作
- Envoy 是一个高性能的代理服务，为服务网格提供了基础。 

#### Kubernetes开放接口 : CRI、CNI、CSI
- CRI（Container Runtime Interface）：容器运行时接口，提供计算资源
- CNI（Container Network Interface）：容器网络接口，提供网络资源
- CSI（Container Storage Interface）：容器存储接口，提供存储资源

  
#### OpenShift 
- 由Red Hat公司开发的开源容器云平台即服务（PaaS）：在Kubernet核心基础上开发 的Web控制台，能执行80％以上的k8s任务。实现更快的应用程序开发，轻松部署和扩展。
- 它是用Go和AngularJS编写的，并且有Apache许可证。

#### IaaS、PaaS 和 SaaS 的区别
IaaS、PaaS、SaaS简单的说都属于云计算+服务。
- **IaaS（Infrastructure as a service – 基础设施即服务）** 
  - 把IT基础设施作为一种服务通过网络对外提供。云提供商有：Amazon AWS、Google Compute Engine、Microsoft Azure、阿里云等
- **PaaS（Platform as a service – 平台即服务）**：
  - 将软件研发的平台作为一种服务。云提供者有：Google App Engine、Microsoft Azure 等
- **SaaS（Software as a Service – 软件即服务）**：
  - 通过网络提供软件服务。

#### 标准HTTP状态码
- **200** OK 服务器返回用户请求的数据，该操作是幂等的
- **201** CREATED 新建或者修改数据成功
- **204** NOT CONTENT 删除数据成功
- **400** BAD REQUEST 用户发出的请求有问题，该操作是幂等的
- **401** Unauthoried 表示用户没有认证，无法进行操作
- **403** Forbidden 用户访问是被禁止的
- **422** Unprocesable Entity 当创建一个对象时，发生一个验证错误
- **500** INTERNAL SERVER ERROR 服务器内部错误，用户将无法判断发出的请求是否成功
- **503** Service Unavailable 服务不可用状态，多半是因为服务器问题，例如CPU占用率大。


#### MQ 介绍
##### 消息队列是一种 **先进先出** 的数据结构，有以下开源组件：
- ActiveMQ : 延迟 ms 级，最早出现，Java语言，单机吞吐量1万
- RabbitMQ : 延迟 us 级，erlang 语言，单机吞吐量1万
- RocketMQ : 延迟 ms 级，功能比较全，Java语言，单机吞吐量10万
- Kafka ： 延迟 ms 级，大数据领域使用较多，Scala 语言，单机吞吐量10万

##### 消息队列通信模式：
- 点对点模式(queue)
  - 生产者把消息发送到 queue，然后消费者从 queue 中取出并消费。
  - 一条消息被消费后，queue中就没有这条消息了，不存在重复消费。
- 发布/订阅模式(topic)
  - 消息生产者将消息放发布到 topic 中，同时可以有多个消息消费者（订阅）消费该消息。

##### 为什么要用 MQ
- **应用解耦：**  
两个服务 Producer 和 Consumer 要做数据传输：传统的作法是使用 RPC 远程调用，但这样会导致两个服务之间的耦合度非常高；
这时我们可以把Producer要发送的数据以消息的形式发送到 MQ，然后 Consumer 监听 MQ ，如果有新的消息就拿出来消费，此时应用就解耦了。
- **流量削峰**  
应用系统如果遇到请求流量瞬间猛增，有可能会将系统压垮；有了消息队列可以将大量请求缓存起来分散到很长一段时间去处理，这样可以大大提高系统的稳定性和用户体验。
- **数据分发**  
通过消息队列可以让数据在多个系统之间进行流通，数据生产方不需要关心谁来使用数据，只要将数据发送到消息队列，数据使用方在消息队列中直接获取数据即可。
##### Kafka 架构介绍
![image](images/C5EE858428E04084826CEF3663EFE77C.png)
- **Producer**：生产者，生产消息，是消息的入口。
- **Kafka Cluster**：Kafka 集群，一台或多台服务器组成。
  - **Broker**：Kafka 的实例节点，每个 Broker 都有一个**不重复**的编号。
  - **Topic**：消息主题，可以理解为消息的分类，数据保存在 topic 中
  - **Partition**：Topic的分区
    - 每个 topic 可以有多个分区，可以**提高负载和吞吐量**。
    - 同一个 topic 在不同分区的数据是不重复的。
  - **Replication**：`Leader`、`Follower`：每个分区都有多个副本，作用是备份。
    - 如果 Leader 出现故障，会由选择某个 Follower 顶上，升级为 Leader。
    - Leader 和 Follower 绝对是不在同一个 Broker 上的
- **Consumer**：消息者，消费消息，是消息的出口。
  - Consumer Group： 可以将多个消费者组成一个消费者组。可以提高吞吐量。

##### Kafka 消息发送工作流程
![image](images/3324B7FD70AB4CE9B23FD3D999D442CA.png)
1. 生产者从 Kafka 集群中获取分区 leader 信息
2. 生产者将消息发送给 leader
3. leader 将消息写入本地磁盘
4. follower 主动从 leader 拉取消息数据写入本地磁盘
5. follower 将消息写入本地磁盘后，向 leader 发送 ACK
6. leader 收到所有 follower 的 ACK 后，leader 向生产者发送 ACK

##### Kafka分区选择模式（3种）
1. 直接指定分区
2. 指定 key, 根据 key 做 hash 然后决定写哪个分区
3. 轮询方式

##### Kafka 发送数据模式（3种）
1. 0 ： 把数据发到 leader 就成功，效率高，安全性最低。
2. 1 ： 把数据发送到 leader , leader 成功就返回 ACK
3. all ： 把数据发送到 leader ，并且 follower 都回应 ACK ，leader 再回应 ACK，安全性最高。


#### OSI七层网络模型 （应表会传网数物）
- **应用层**
  - 用户接口，比如各类软件。
  - 协议有：HTTP HTTPS FTP TFTP SMTP SNMP POP3 DHCP DNS TELNET 等
- **表示层**
  - 数据的表示、安全加密、压缩。
- **会话层**
  - 主机进程会话：建立、管理、终止。
- **传输层**
  - 传输数据的TCP/UDP协议和端口号，传输前的差错校验和流控。
- **网络层**
  - 逻辑地址寻址，网络路径选择。
  - 协议有：ICMP IGMP IP（IPV4 IPV6） 
- **数据链路层**
  - 将`比特`(01位)组合成字节再组合成帧，用MAC地址访问媒介，错误检验与修正。
- **物理层**
  - 物理接口，比特流传输，电气特性。
#### TCP/IP四层模型：网络协议
- 应用层 ：HTTP HTTPS FTP TFTP SMTP SNMP POP3 DHCP DNS TELNET
- 传输层 ：TCP, UDP
- 网络层 ：IP, ICMP, ARP, RARP, AKP, UUCP
- 数据链路层：
  - FDDI, Ethernet, Arpanet, PDN, SLIP, PPP
  - IEEE 802.1A, IEEE 802.2到IEEE 802.11



## 学习补充
- 维护分布式文件存储 Ceph, 对存储资源做统一管理
- 熟悉DevOps流程，参与 CI/CD 在容器化场景中的流程设计和开发
- 基于应用场景对原生 Kubernetes 进行二次开发, 包括 CNI, CSI, CRD和 Operator 设计模式
- Operator、Istio等容器和云原生技术
- 熟悉K8s和Docker的网络解决方案，具有SDN网络调研能力优先。
- 熟悉 Jenkins/ELK/Prometheus 等技术优先；
- 熟悉常用数据结构和算法，熟悉主流分布式系统设计概念和算法
- 熟练使用MySQL/MongoDB/Redis等数据库编程以及HDFS文件系统。
- 有Linux Kernel核心子系统，如内存管理，文件系统，网络，进程调度，Cgroup经验等更好；
- 精通 Golang 语言，有 Golang 实际项目代码经验。
- 维护 Kubernetes 集群, 熟悉 rbac 权限管理, 分配,隔离各业务所使用的集群资源






## 自我描述：责任心

- 较强的团队沟通和协作能力，较强的自我驱动能力。
- 有较好的逻辑思维能力，较强的抽象、概括、总结能力；
- 有较好的沟通交流能力，善于主动思考和行动，乐于解决具有挑战性的问题，对待技术有强烈兴趣；
- 对技术有追求
- 具备较强的沟通表达能力、较强的服务意识；
- 对技术仍保持热情，为人积极，沟通理解能力强；
- 具有强烈的责任心，能够积极跟进问题；
- 思路清晰，能独立解决问题，有较强的学习能力、沟通能力及团队合作精神。
- 有作为全栈工程师的系统运维经验
- 有较强的业务抽象、逻辑分析能力，善于总结沉淀；
- 善于发现问题，解决问题，并能落地解决方案；
- 良好的沟通能力和团队协作能力
- 具有较强学习能力，分析解决问题能力，强烈的责任心和团队合作能力
- 良好的编码习惯，严格遵守团队开发规则和流程。
- 重视代码质量，代码可维护性，有编写单元测试用例习惯。
- 对前缘技术有深厚热爱，能承受一定工作压力。
- 良好的职业素养和自驱力，优秀的技术热情，能够持续学习和自我提高；
- 积极乐观，认真负责，善于协作；
- 具有很强的发现问题、分析问题和解决问题的能力
- 具有较好的逻辑思维分析和沟通表达能力，对新事物具有探求精神，思维活跃，善于学习，具备较强的团队协作意识
- 踏实靠谱，主动性和自我驱动意识强，责任心强；热衷于钻研业界前沿技术可加分；
- 较强的沟通能力和逻辑能力，具备良好的团队合作精神和主动意识，良好的自我驱动和学习能力
- 诚实守信、作风踏实严谨、责任心强，具备良好团队协作能力精神，学习能力强，善于解决复杂问题；
- 具有良好的编码习惯，不断完善已有代码；
- 热爱技术，喜欢钻研，善于解决问题。
- 学习能力强，肯钻研，认真踏实
- 有良好的时间观念，善于团队协作，项目管理，主动思考，自我驱动力强，乐于分享
- 对产品有强烈的责任心，具备良好的沟通能力和协作能力。
- 具有清晰的系统思维逻辑，对系统设计与核心部件开发以及新技术追踪有浓厚兴趣；
- 良好的沟通和分析问题与解决问题能力；
- 诚实、踏实、积极主动、抗压能力强，喜欢挑战困难
- 从事云计算行业研发者、对容器技术有浓厚兴趣优先
- 良好的沟通能力和团队协作精神，严谨的工作态度与高质量意识
- 善于学习新的知识，动手能力强，有强烈的责任心，喜欢钻研技术
- 热爱编程，学习能力强，有强烈的求知欲、好奇心和进取心 ，能及时关注和学习业界最新技术。
- 思维逻辑清晰，抗压能力强，良好的沟通协调能力，有责任心，有工作目标感与计划性，适应互联网行业的高速节奏。
- 有良好的团队协作精神，富有责任心，能承受一定强度的工作压力
- 具有良好的沟通能力，思路清晰，善于思考，能独立分析和解决问题；
- 有责任感，对技术有执着追求；
- 踏实靠谱；主动性和自我驱动意识强；热衷于钻研业界前沿技术。
- 具备较强的学习能力和独立分析解决问题的能力，责任心强，自我驱动，有良好的团队合作精神；
- 具备强烈的技术热情和钻研精神，热爱新技术，追求细节和极致，有解决各类疑难问题的强烈愿望；
- 有良好的沟通能力，注重团队协作，善于主动思考和行动。
- 对新技术充满好奇和学习激情，对技术趋势敏感；
- 对改善开发者感受和产出充满激情。
- 对新技术有执着追求，热爱编程。善于抽象、总结、思考，能及时关注和学习业界新技>术；
- 有较强的自学能力和钻研精神，具有良好的沟通能力和团队合作能力，综合能力强
- 责任心强，良好的沟通能力、学习能力，能独立解决问题，抗压能力强；
- 踏实靠谱；主动性和自我驱动意识强；
- 热衷于钻研业界前沿技术。