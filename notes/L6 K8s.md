# Part 1: Container Orchestration and Kubernetes

## What is Container Orchestration?

![[Pasted image 20260425213301.png]]
- **核心概念**：随着 Docker 容器的发展，系统需要管理容器的生命周期、处理它们之间的通信，并将其分布在不同的计算节点上 。
- **云端操作系统**：就像操作系统（如 macOS、Linux）管理单台计算机上的内存和文件一样，容器编排工具负责管理跨多个云端计算节点的服务、容器和卷等资源 。
    
---
### Declarative Application Management

- **声明式应用管理 (Declarative Application Management)**：通过声明系统的“期望状态”来管理软件，而不是输入一系列具体的执行命令 。这种方式让基础设施具备“自我修复”能力（例如节点崩溃时自动恢复），并且具有“幂等性”，即重复应用相同配置不会改变系统状态，从而减少错误 。期望状态通常用 YAML 或 JSON 配置文件来表达 。
	![[Pasted image 20260425213336.png]]
    
- **基础设施即代码 (Infrastructure as Code, IaC)**：使用代码（主要是配置文件）而非手动点击图形界面来管理基础设施 。这便于使用 Git 等工具追踪架构变更，并能清晰地记录系统架构 。
	![[Pasted image 20260425213438.png]]
    

### Compose vs Swarm vs Kubernetes
![[Pasted image 20260425213516.png]]
- **Docker Compose**：用于定义和管理运行在**单一计算节点**上的一组协同工作的容器 。
    
- **Docker Swarm**：将 Compose 的能力扩展到了**多个计算节点** 。
    
- **Kubernetes (K8s)**：行业标准，最初由 Google 开发 。它除了具备跨节点管理容器的核心功能外，还提供了强大的抽象功能和工具集来构建复杂系统 。

---

## Kubernetes (k8s)

### Kubernetes on the Cloud
- **商业云托管服务**：主流云厂商均提供 k8s 托管服务，如 AWS 的 EKS、Azure 的 AKS 和 Google Cloud 的 GKE。
- **私有云部署**：支持在自有资源上运行私有集群（例如基于 OpenStack 的墨尔本研究云 MRC）。
	
- **跨云可移植性**：除了底层资源分配方式的不同（如 MRC 使用 OpenStack，AWS 使用 EC2），一旦资源就绪，不同平台上的 k8s 集群行为基本一致，极大地方便了应用的跨云迁移（除非使用了特定云厂商的专有服务如 AWS Aurora）。
    

### Kubernetes Architecture
![[Pasted image 20260426132247.png]]
- **控制平面 (Control Plane / Master Nodes)**：负责维护集群的期望状态。具体职责包括调度容器、存储集群配置、管理服务以及暴露 k8s API（通常运行在 6443 端口，供 CLI 命令行工具交互调用）。
    
- **数据平面 (Data Plane / Worker Nodes)**：负责实际运行用户的工作负载。它提供工作负载 Pod 运行所需的执行环境、网络和存储机制。
    

### What K8s Does for Software Application
![[Pasted image 20260426132725.png]]
- **高可用性保障**：确保应用容器持续运行。如果某个容器停止，它会被重新生成；如果某个节点宕机，该节点上的容器会被自动迁移到存活的节点上。
    
- **服务发现与内部 DNS**：为服务分配稳定的 DNS 名称（如 `pg.dev.svc.cluster.local`），避免依赖随时可能改变的 IP 地址。
    
- **智能调度**：可根据需求自动将容器分配给特定的节点（例如，将机器学习容器专门分配给带有 GPU 或拥有 16GB RAM 的节点）。
    
- **负载均衡**：自动将容器的计算负载分散到集群的不同节点上。
    
- **环境隔离**：通过命名空间 (Namespaces) 机制，可以在同一个集群中托管同一容器的不同版本（例如隔离 `dev` 开发环境和 `prod` 生产环境）。
    
- **配置管理**：提供向容器分发配置数据和设置环境变量的内置机制。

---

## Kubernetes objects (Pods, Services, Ports, etc)

## Concepts
![[Pasted image 20260426133347.png]]

- **Node (节点)**：运行 k8s 的计算节点（通常是虚拟机）。
    
- **Persistent Volume (持久卷, PV)**：可以附加到节点并作为文件系统挂载的持久化存储。
    
- **Pod (容器组)**：k8s 中最小的可部署单元。它由一个或多个为了完成特定任务而协同工作的容器组成。
    
- **Deployment (部署)**：管理一组同时运行的相同 Pod。主要用于维护同一应用程序的多个副本（例如，同时维持 3 个 Nginx Pod 的运行）。
    
- **Service (服务)**：在 k8s 集群**内部**，将 Pod 提供的功能暴露在特定的网络端口上。
    
- **Ingress (入口)**：允许集群**外部**（非集群托管）的客户端访问集群内的一个或多个 Service。支持基于主机名 (hostname) 和路径 (paths) 的路由转发。
    
- **ConfigMap (配置字典)**：一种以集中化方式向 Pod 传递普通参数/配置的机制。
    
- **Secrets (密钥)**：一种以集中化方式向 Pod 传递敏感参数（如密码、证书）的机制。
    
- **Namespace (命名空间)**：为了方便管理，将上述所有的 k8s 对象（除了 Node 和 Persistent Volume 这种集群级别资源）在逻辑上进行分组的隔离空间。

---

## How Kubernetes Components Fit Together
![[Pasted image 20260426135434.png]]
1.  **外部请求进入**：`client` 发起请求 $\rightarrow$ 命中 `inapi` (Ingress)。
2.  **集群内部分发**：`inapi` $\rightarrow$ 转发至 `restapi` (Service)。
3. **负载均衡到后端**：`restapi` $\rightarrow$ 将请求分发给 `web` 部署下的某一个健康 Pod (`ws-0` 或 `ws-1`)。
4.  **跨命名空间访问数据库**：Web Pod 在处理请求时如果需要查阅数据 $\rightarrow$ 通过集群内部 DNS 呼叫 `db` (Service) $\rightarrow$ 最终访问到 `mysql` 容器获取数据。

---

## Pods
![[Pasted image 20260426141902.png]]
- **基本定义**：虽然 Pod 通常只包含单个容器，但在需要多个容器协同工作的场景下，Pod 提供了一个非常实用的架构抽象。
    
- **资源共享**：同一个 Pod 内的所有容器**必须运行在同一个计算节点上**，并且它们天然**共享存储卷和网络空间**。
    
- **清单文件 (Manifest)**：在声明 Pod 时，必须指定 Pod 名称、容器名称以及容器镜像。此外，还可以配置环境变量、容器暴露的端口以及需要挂载的存储卷等。
	
- **Scheduling**: 
	- **亲和性与反亲和性 (Affinity & Anti-affinity)**：明确规定某个 Pod 倾向于运行在哪些类型的节点上，或者绝对不应该运行在哪些节点上。
	- **其他高阶控制**：除了亲和性，还可以使用节点选择器 (Node selector)、污点 (Taint) 与容忍度 (Tolerations) 等机制进行精细化的调度管理。
    
- **镜像管理**：K8s 会自动下载并运行配置文件中声明的容器镜像。如果使用的是私有镜像仓库，则必须提前在集群中配置并存储好相关的访问凭证。
    

### Why Pods?
Pod 的设计非常适合构建基于微服务 (Microservices) 的**松耦合架构 (Loosely-coupled architecture)**，它允许将复杂的应用拆分成多个独立的服务组件 。

![[Pasted image 20260426150336.png]]

- **日志解耦处理 (Logging)**：与其将日志处理逻辑（例如将日志发送到 ElasticSearch）直接写死在主应用代码中，更好的架构方案是让应用直接将内容输出到标准输出 。然后，在同一个 Pod 内运行另一个专门的容器来收集这些文件变动、处理并将日志发送到目标位置 。这样，如果未来需要更改日志的发送目的地，主应用程序的代码完全不需要改动 。
    
- **Init 容器 (初始化模式)**：Pod 可以包含一个 `init` 容器，主要用于在主应用容器启动之前执行“预置/准备”任务 。例如，在启动数据库容器之前，先运行一个 `init` 容器来自动创建默认数据库，并填充必要的表结构和初始用户数据 。
    
- **流量与安全代理**：可以在主应用容器旁运行一个辅助容器，专门用于处理或丰富网络请求的输出 例如，统一为输出添加特定的 HTTP 头，或者为了提高安全性而自动剥离某些敏感的 HTTP 头 
    
- **Sidecar (边车模式)**：上述这种伴随主应用程序容器运行、用于扩展或增强主应用功能的辅助容器，在 Kubernetes 架构中被统称为 **Sidecars**

---

## Deployments
在实际的 Kubernetes 工程中，我们极少直接创建独立的 Pod，而是通过 **Deployments (部署)** 来声明和管理它们。

**YAML 清单核心解析 (以 WordPress 部署为例)：**
![[Pasted image 20260426160048.png]]
- **元数据与声明**：
    - `apiVersion: apps/v1` 和 `kind: Deployment`：声明这是一个部署对象。
    - `replicas: 1`：定义期望同时运行的 Pod 副本数量。
        
- **Pod 模板 (`template`)**：定义了 Deployment 要创建的 Pod 长什么样。
		
    - **labels**：例如 `app.kubernetes.io/name: wordpress`。这是极其关键的一步，用于给 Pod 打上标记，后续的 Service 就是靠这个标签来找到它们的。
	    
    - **容器组 (`containers`)**：声明容器的镜像来源（如 `docker.io/bitnami/wordpress`）以及启动所需的**环境变量 (`env`)**（如设定数据库名称）。
        
	- **调度策略 (`affinity`)**：
	    - **节点亲和性 (Node Affinity)**：通过 `matchExpressions` 定义强制性的调度规则。例如，要求该应用必须被调度到含有 `magnum.openstack.org/nodegroup = default-worker` 标签的节点组上。

---

## Services

由于 Pod 是可以被随时销毁和重建的（IP 会变化），我们需要 **Service (服务)** 来为其提供一个固定的内部网络访问端点。
![[Pasted image 20260426160703.png]]
**YAML 清单核心解析：**

- **元数据与声明**：
    - `apiVersion: v1` 和 `kind: Service`：声明这是一个服务对象。
    - `type: ClusterIP`：这是默认类型，表示该服务仅在 k8s 集群**内部**可访问。
        
- **端口映射 (`ports`)**：
    - `port: 80`：Service 自身向外（集群内部其他组件）暴露的端口。
    - `targetPort: 8080`：流量最终将被转发到的后端 Pod（容器）实际监听的端口。
        
- **标签选择器 (`selector`)**：
    - **核心机制**：通过匹配标签 `app.kubernetes.io/name: wordpress`，Service 能够自动发现所有带有此标签的 Pods，并将网络流量准确地路由给它们。这就是 K8s 中实现松耦合的关键所在。

---

## Ingress Routing

Ingress 主要用于将集群外部的 HTTP/HTTPS 流量灵活地路由到集群内部的 Service 。

![[Pasted image 20260426162504.png]]
- **路由规则**：通过匹配请求的主机名或路径 (Paths) 来决定流量的去向。
    
- **配置注意点**：在定义 Ingress 规则时，必须明确指定后端 Service 的**名称**及其**目标端口号**（因为一个 Service 可能会暴露多个不同的端口）。
    
---

## Persistent Volumes

在 K8s 工程实践中，开发者通常不会直接去创建持久卷 (PV)，而是通过**持久卷声明 (PVC, Persistent Volume Claim)** 来向集群动态申请存储资源 。

![[Pasted image 20260426162546.png]]
1. **申请存储 (PVC)**：
    - 在 PVC 清单中，声明所需的存储类 (`storageClassName`)、所需的容量（如 `400Gi`）以及访问模式（例如 `ReadWriteOnce` 表示该卷只能被单个节点以读写模式挂载）。
        
2. **在 Pod 中挂载**：
    - 首先在 Pod 的 `volumes` 层级中，通过 `claimName` 引用刚才创建的 PVC 。
        
    - 然后在具体的 `containers` 层级中，使用 `volumeMounts` 将该卷挂载到容器文件系统中的指定路径（例如 `/data`）。
        

## Block Storage is not Object Storage

理解底层存储的区别对于云原生架构设计非常关键。
![[Pasted image 20260426162700.png]]

| **特性**     | **块存储 (Block Storage)**             | **对象存储 (Object Storage)**                |
| ---------- | ----------------------------------- | ---------------------------------------- |
| **工作原理**   | 类似于物理磁盘驱动器，直接附加挂载到容器上使用 。           | 基于 HTTP 协议的存储服务 Example: OpenStack swift |
| **性能特点**   | 速度快、延迟低，专门针对**大量且频繁的读/写操作**进行了优化 。  | 延迟相对较高，但**吞吐量极大** 。                      |
| **共享与挂载**  | 默认情况下通常只附加到单个节点上（`ReadWriteOnce`） 。 | 可以轻松跨越多个不同的 Pod 甚至集群进行共享读写 。             |
| **最佳使用场景** | 作为数据库的数据盘等核心业务存储。                   | 存储大型、不可变的数据对象，例如**数据库备份归档** 。            |

---

## ConfigMaps

在 Pod 的清单文件中直接硬编码环境变量虽然方便，但不利于配置的集中管理和复用。**ConfigMaps** 提供了一种更好的集中化参数传递方式 。

- **作用域**：可以将其看作是供 Pod 访问的字典，但它**仅限同一个命名空间 (Namespace)** 内的 Pod 访问，跨命名空间无法读取 。
    
- **使用方式**：创建后，可以在 Deployment 清单中将 ConfigMap 作为**环境变量 (Environment Variables)** 引入，或者作为**文件卷 (Mounted Files)** 挂载到容器中 。
    
![[Pasted image 20260426182830.png]]

```YAML
# 1. 声明一个 ConfigMap (存储数据库基础路径)
kind: ConfigMap
apiVersion: v1
metadata:
  name: dbparameters
data:
  basedir: '/opt/bitnami/mariadb'
---
# 2. 在 Pod/Deployment 中作为环境变量引用
env:
  - name: BASEDIR
    valueFrom:
      configMapKeyRef:
        name: dbparameters
        key: basedir
```


```YAML
# 1. 声明一个存储脚本的 ConfigMap
kind: ConfigMap
apiVersion: v1
metadata:
  name: warmer-script
data:
  warmup-script.sh: |
    #!/bin/sh
    set -e
---
# 2. 在 Pod/Deployment 中作为文件挂载
volumeMounts:
  - name: script-vol
    mountPath: /scripts
volumes:
  - name: script-vol
    configMap:
      name: warmer-script
```

## Secrets

虽然 ConfigMaps 很好用，但它以明文形式存储数据，并不安全 。对于密码、Token 和证书等敏感信息，应该使用 **Secrets**。
![[Pasted image 20260426183019.png]]
- **安全特性**：
    
    - 可以配置为加密存储（虽然 K8s 默认不开启加密，需额外配置工具） 。
    - 它的具体内容**不会显示在 K8s API 的常规日志中** 。
    - 可以设置更严格的访问控制权限 。
    - 在传输到 Pod 的过程中是加密的 。
        
- **存储机制**：默认情况下，集群中的 Secrets 并没有被加密，而是使用了 **Base64 编码** 。
    
- **使用方式**：与 ConfigMaps 非常相似，也是通过环境变量或挂载卷引入，并且同样**只能被同一命名空间下的 Pod 访问** 。
    

**解码与引用示例：**

1. **本地解码查看**（例如查看 ElasticSearch 密码）：

```Bash
kubectl get secret elasticsearch-es-elastic-user -n elastic -o jsonpath='{.data.elastic}' | base64 -d
```
    
	(注：该命令通过 JSONPath 提取密码数据，并通过 `base64 -d` 进行解码还原 )
    
2. **在清单中作为环境变量引用**：

```YAML
env:
 - name: ESPASSWORD
    valueFrom:
         secretKeyRef:
         name: my-secret
	     key: password
```

---

# Part 2: Kubernetes on MRC 
## Kubernetes at the Melbourne Research Cloud
![[Pasted image 20260426192749.png]]
- **基础设施依赖**：Kubernetes 的计算节点、磁盘卷和网络资源均依赖于底层的云基础设施 。
    
- **OpenStack Magnum 工具**：虽然在 MRC 上运行 K8s 集群有多种途径，但最简单、推荐的方式是使用名为 **Magnum** 的 OpenStack 组件 。
    
- **Magnum 的作用**：
    
    - 它会与其他 OpenStack 组件交互，负责分配所需的云资源，并将 Kubernetes 控制平面 (Control Plane) 自动部署到集群节点上 。
        
    - 它还可以用来动态调整集群规模（例如添加或移除计算节点） 。
        

## Access to an MRC K8s Cluster

![[Pasted image 20260426192925.png]]
- **安全限制**：为了保障云基础设施的安全，MRC 实施了严格的安全限制，这使得直接访问集群变得相对复杂 。
    
- **访问工具**：日常交互主要使用 CLI 命令行客户端（如 `kubectl` 和 `openstack` 工具） 。
    
- **端口转发 (Port Forwarding)**：为了访问集群内托管的服务（如 HTTP 接口端点），**必须且只能使用 K8s 的端口转发机制** 。
    
- **安全认证底层**：端口转发机制是建立在 6443 端口的 Kubernetes API 之上的，系统仅允许持有正确 SSL 证书的客户端接入 。
    

## Where to put the kubeconfig File?
![[Pasted image 20260426193229.png]]
- **核心作用**：集群的身份验证完全依赖于 `kubeconfig` 文件。该文件由团队中负责执行集群资源分配的那**唯一一名成员**生成 。
    
- **文件内容**：该文件包含了连接集群所需的所有核心机密信息，包括：
    
    - Master 节点的浮点 IP 地址 (floating point address) 。
        
    - 用于认证本地客户端终端（你的笔记本电脑）的专属客户端 SSL 证书 。
        
    - k8s API 监听的端口号（所有通信均通过 HTTPS 加密） 以及其他参数 。
        
- **团队分发**：生成完毕后，必须将其分发给团队中的所有成员，以便大家都能在各自的电脑上连接并操作集群 。
    
- **🚨 绝对安全红线**：**严禁**将 `kubeconfig` 文件放置在公开或半公开的位置（如 GitLab 代码仓库、Ed 讨论区、OneDrive 或 Google Drive），以防被恶意使用者窃取集群控制权 。

 ---

# Part 3: Hands-on Kubernetes

## Kubernetes Tools

![[Pasted image 20260426201642.png]]
- **Magnum**: OpenStack 的组件，用于从云基础设施中分配资源，并包含向 OpenStack 云发送命令的客户端工具。
    
- **Kubectl**: 核心命令行工具，用于向 K8s 集群 API 发送指令，管理集群内部资源。
    
- **Helm**: K8s 的命令行软件部署工具（包管理器），可以把它理解为 K8s 界的 `npm` 或 `pip`。
    
- **Minikube**: 在单台机器（如笔记本电脑）上构建迷你 K8s 集群的工具，非常适合本地开发和学习。
    
---
## Magnum

Magnum 使用“集群模板 (Cluster Templates)”来分配创建 K8s 集群所需的资源。模板本质上是一个声明了节点数量、基础镜像、网络等需求的清单文件。

- **命令语法**：使用**对象-动作 (object-action)** 语法。命令前缀为 `openstack coe` (COE 代表 Container Orchestration Engine)。
    
![[Pasted image 20260426202612.png]]

```Bash
openstack coe cluster create   # 创建集群
openstack coe cluster list     # 列出集群
openstack coe cluster show     # 显示集群详情
openstack coe cluster config   # 获取集群配置并写入客户端的 kubeconfig 文件
openstack coe cluster resize   # 调整集群节点数量
openstack coe cluster delete   # 删除集群
```

---

## How to Resize a Magnum Cluster

![[Pasted image 20260426202710.png]]
**增加节点**（以提升容量）：
```Bash
openstack coe cluster resize --nodegroup <node_type> <cluster_name> <desired_number_of_nodes>
```

_注：新增节点后，新建的 Pod 会在一段时间后自动调度到新节点上。_

**安全移除故障节点的标准流程**：

1. **停止调度**：阻止新工作负载部署到故障节点。
```Bash
  kubectl cordon <failing_node_name>
```

2. **添加新节点**：使用上述的 `resize` 命令增加一个新节点。
``` Bash
kubectl drain <failing_node_name>
```

4. **移除节点**：使用节点 ID 彻底移除故障节点。
```Bash
openstack coe cluster resize --nodes-to-remove <failing_node_id> --nodegroup <node_type> <cluster_name> <desired_number_of_nodes>
```

---
##  Kubectl

与 Magnum 不同，Kubectl 使用**动作-对象 (action-object)** 语法。

### 1. 查看与监控资源
![[Pasted image 20260426211354.png]]
- **查看节点与 Pod**：
```Bash
kubectl get nodes                        # 列出所有节点
kubectl get pods -n <namespace>          # 列出指定命名空间下的 Pods
kubectl get pods -A                      # 列出所有命名空间下的 Pods
kubectl get all -n <namespace>           # 列出指定命名空间下的所有资源
kubectl get all -A                       # 列出整个集群的所有资源
```

- **查看详细信息与资源占用**：
```Bash
kubectl describe pods <pod_name> -n <namespace>  # 查看 Pod 详细信息
# 查看节点详细信息（包含内部运行的 Pods、RAM 和 CPU 使用率）               
kubectl describe node <node_id>
kubectl top pods -n <namespace>    # 查看特定命名空间下 Pod 的实时 CPU 和内存占用
```

_(注：内存单位 `Mi` 表示 $2^{20}$ 字节，CPU 单位 `m` (millicores) 表示千分之一的虚拟 CPU)_

![[Pasted image 20260426211623.png]]
### 2. 状态变更与网络转发

- **应用清单 (Manifest)**：
```Bash
kubectl apply -f <manifest>.yaml -n <namespace> --wait
```
	_(注：`--wait` 参数会让命令在应用完成前保持等待状态，直到生效后再返回终端提示符)_
	
- **端口转发 (安全访问内部服务)**：
	无需 Ingress 即可将 Pod 或 Service 的端口安全地映射到本地电脑：
```Bash
kubectl port-forward deployment/<deployment-name> <local-port>:<pod-port>
```
    

### 3. 重启与调试
![[Pasted image 20260426214152.png]]
- **安全重启 (Rollout Restart)**：当配置字典 (ConfigMap) 更新时，建议使用此方法优雅重启 Deployment 中的 Pods（逐个重启，如果失败则停止，保证服务不中断）。
    
```Bash
kubectl rollout restart deployment <deployment_id> -n <namespace>
```
    
- **Dry-run (模拟运行)**：用于生成 YAML 清单而不实际执行更改。
    
```Bash
kubectl create secret generic mysecret --from-literal=pwdes='elastic' --dry-run=client -o yaml
```
    
- **进入容器/节点终端 (Interactive Shell)**：
![[Pasted image 20260426214523.png]]
```Bash
# 进入 Pod 内的容器：
kubectl exec <pod_name> -it --namespace <namespace> -- /bin/bash

# 调试 Node 节点 (创建一个临时 debug pod 并挂载节点文件系统至 /host)：
kubectl debug node/<node_id> -it --image=busybox --profile=general
```
    

## Helm

Helm 提供了一种极其方便的方式来在 K8s 集群上部署应用程序。它使用 **Charts**（定义了集群上需创建资源的清单集合）。
![[Pasted image 20260426214602.png]]
- **支持参数自定义**：通常允许通过 CLI 设置参数来深度定制部署。
    
- **常用核心命令**：
```Bash
helm repo add                     # 添加 Chart 仓库
helm repo update                  # 刷新仓库内容获取最新版本

# 安装或更新 Chart：
helm upgrade --install <release_name> <chart_name> --set <key>=<value>
# 也可以使用 YAML 文件批量传入自定义参数：
helm upgrade --install <release_name> <chart_name> --values <manifest>.yaml
```