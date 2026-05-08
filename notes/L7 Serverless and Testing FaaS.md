## Disambiguation

![[Pasted image 20260501170311.png]]
- **FaaS 与 Serverless 的关系**：FaaS 通常也被称为 Serverless（无服务器计算），后者听起来更吸引人，但精确度稍逊。
    
- **核心理念**：开发软件应用时无需操心底层基础设施（特别是当负载增减时的自动扩缩容问题）。云服务商会替你管理服务器和容量。
    
- **Server-unseen**：因此，它更准确的描述是“服务器不可见”，而不是真的“没有服务器”。
    
- **微服务的极致形态**：FaaS 是微服务架构的一种极端形式。环境允许函数被动态添加、删除、更新、执行以及自动缩放。
    
- **内存按需加载**：在 FaaS 环境中，各个函数是互相独立执行的，并且**只有在需要时**才会被加载到主内存中。


## Why Functions? 

![[Pasted image 20260501170338.png]]
- **计算机科学中的定义**：一段接收参数并返回值的代码。
	
- **三大理想特性**：函数应该具备以下三个特性，这使得它们成为并行执行和快速扩缩容的完美选择：
    - **Free of side-effects** (无副作用)
    - **Ephemeral** (生命周期短暂)
    - **Stateless** (无状态)

## Functions? 
![[Pasted image 20260501170414.png]]
- FaaS 语境下的函数，与普通编程语言中的函数有着本质区别。
    
	- **编程语言的函数（如 Python）**：是一组在**同一个进程内**执行并可能返回对象的语句。
	    
	- **FaaS 函数**：是一个**独立的进程**，它返回一个值（通常以 JSON 格式表示），供其他进程消费。
    
- **底层形态**：在许多 FaaS 框架中，一个 FaaS 函数实际上就是一个 **Docker 容器**。
    

> **生活化比喻**：普通 Python 函数像是同一个车间里的不同流水线工位，互相之间共享车间环境；而 FaaS 函数就像是已经打包好的独立集装箱（Docker容器），随时可以搬运到任何码头独立运作，并且产出标准化的货物（JSON数据）。


## Why FaaS? 
![[Pasted image 20260501170443.png]]
- **部署更简单**：云供应商接管了所有基础设施工作。
    
- **计算成本降低**：资源利用极其高效，**仅针对函数真正执行的时间段进行计费**。
    
- **降低复杂性与提高灵活性**：系统内在的松耦合架构使得应用更易于维护和调整。
    

## FaaS Applications? 
![[Pasted image 20260501170514.png]]
- **事件驱动**：函数是由“事件 (Events)”触发的。
    
- **组合与复用**：函数可以互相调用（函数组合），并且只要写得足够通用，就可以被随意复用。
    
- **构建应用软件**：将事件场景与函数结合起来，就构建出了完整的软件应用。
    
- **典型的触发器例子**：
    - 每小时触发一次（例如：压缩日志文件）
    - 每次向集群添加新节点时触发
    - 当 GitHub 中的 Pull-request 被合并时触发
    - 当队列中存储了新消息时触发
        
- **UI 开发模式的延伸**：这种事件驱动+函数的组合方式，非常类似于构建用户界面 (UI) 软件的方式——用户的具体动作（点击）触发了某段代码的执行。
    

## FaaS Services and Frameworks 
![[Pasted image 20260501170552.png]]
- **优势**：允许函数直接使用各自云平台的专属丰富服务。
- **开源框架 (Funtainers)**：例如 Apache OpenWhisk、OpenFaas、Fission、Fn 和 Knative（Funtainers 即 functions containers 的缩写）。
    
- **核心区别**：专有 FaaS 服务属于商业闭源，而开源 FaaS 框架可以部署在你**自己的服务器集群**上，你可以亲自深入查看底层原理、拆解并改进它们。

---

# Part 2: More About Functions

## Ephemeral Functions
![[Pasted image 20260501174143.png]]
- 短暂存在的函数是指其创建、执行和移除只需很短时间的函数。
    

## Side-effect-free Functions
![[Pasted image 20260501174153.png]]
- 不修改软件应用程序状态的函数被称为side effect free function（例如，接收一张图片并返回该图片缩略图的函数）。
	    
	- 改变应用程序状态的函数不是side effect free（例如，返回图片缩略图并将其保存到数据库的函数）。
	- side effect free function可以并行运行，而无需过多担心并发问题（例如多个函数同时写入同一个文件）。
	    
- 在non-trivial system中，side effect是不可避免的。因此，必须考虑如何使带有side effect的函数并行运行（这在 FaaS 环境中通常是必需的），同时避免资源冲突。
    
- 良好的实践是将具有side effect（非无副作用）的函数数量限制在最低限度，而不是将改变状态的代码片段分散在整个软件应用程序的众多函数中。
    

## Stateful/Stateless Functions
![[Pasted image 20260501174253.png]]
- 函数的一个重要子集是由有状态函数组成的。
    ![[Pasted image 20260501174504.png]]

- Stateful 和 Stateless 的核心区别确实在于**内部是否保留记忆（存储信息）**：
	- **Stateful (有状态)**：记忆保存在函数内部。函数执行完毕后，数据依然留存在这个函数实例的内存里。当同一个请求再次到达时，输出会受到内部已存信息的影响。（例如：将商品保存在函数自己内存中的“购物车”里）。
	
	- **Stateless (无状态)**：函数内部绝对不存储信息。为了实现业务逻辑，无状态函数会将需要记忆的信息**写入到外部的数据库 (DBMS) 或磁盘中**。（例如：将商品信息写入到外部数据库的“购物车”表中）。
    
- 在 FaaS 环境中，同一个函数可能会同时存在多个实例，云平台无法保证同一个用户的两次请求会被分配给同一个函数实例。如果你把数据存在函数实例 A 的内存里（有状态），下一次请求被分配到实例 B 时，数据就丢失了。 因此，FaaS 架构通常要求我们将函数设计为**无状态的**，把“记忆”的工作全部交给专门的外部存储服务。
    
- 既无状态又无副作用的函数被称为纯函数（**其输出仅取决于其输入**）。
    

## Synchronous/Asynchronous Functions
![[Pasted image 20260501174547.png]]
- 大多数 FaaS 函数是同步的，因此它们会立即（或几乎立即）返回结果。
    
- 然而，有些函数可能需要较长时间才能返回结果，因此它们在此过程中会引发超时并锁定与客户端的连接，因此最好将它们转换为异步函数。
    
- 异步函数返回一个代码，告知客户端执行已开始（通常是 HTTP 状态码 202），然后在执行完成时触发一个事件。
    
- 在更复杂的情况下，可以使用涉及消息队列系统的发布/订阅模式来处理异步函数。

----

# Part 3: Monolithic vs FaaS Applications

## The Requirements

- 假设我们需要开发一个满足以下要求的应用程序：
    - 从气象局 (BoM) 站点抓取天气数据。
    - 从空气质量监测服务抓取污染数据。
    - 将数据存储在 ElasticSearch 中。
    - 计算某一天所有站点的平均温度。
    - 计算某一天单个站点的平均温度。
        
## Monolithic Application

![[Pasted image 20260501184150.png]]
- 让我们看看“标准”的 Web 应用程序如何满足这些需求：
    - 定时器调用 API 的抓取端点，触发从 BoM 和空气质量服务收集数据。
    - 将数据转换为通用格式。
    - 将转换后的数据插入 ElasticSearch。
    - 另一个端点允许客户端应用程序请求平均温度（使用 ElasticSearch 查询计算得出）。
        

## Monolithic Application?

![[Pasted image 20260501184319.png]]
- 该应用程序可以很容易地进行垂直扩展（使用更强大的虚拟机），但无法进行水平扩展（到集群中的其他节点）。
    
- 该应用程序的大多数（如果不是全部）模块同时加载到内存中，导致使用的资源超过了严格所需的资源。
    
- 如果应用程序的某个模块发生严重故障（例如 Java 中的内存溢出错误），则会导致整个应用程序崩溃。
    
- 应用程序的所有模块都是紧密耦合的（很可能使用同一种编程语言编写），因此对一部分代码的更改（例如，使用较新版本的库）可能会引发一系列其他更改。
    

## Serverless Version of Application
![[Pasted image 20260501184432.png]]
- 每个功能由不同的（无状态）函数提供。
    
- 大多数函数甚至不直接相互“交谈”，而是通过队列中的异步消息进行通信。
    
- 函数仅在需要时加载。
    
- 发生故障的函数不会导致整个应用程序崩溃。
    
- 同一函数的多个副本可以分布在不同的节点上，以实现水平扩展。
    
- 函数可以用不同的语言编写，并且仍然可以协同工作。
    
- 函数可以使用不同的环境（例如 Python 3.9 和 Python 3.10），并且仍然可以协同工作。
    
- 该应用程序现在更容易修改，并且更加健壮。

---

# Part 4: Fission FaaS

## Fission
![[Pasted image 20260501201036.png]]
- Fission ([https://fission.io/](https://fission.io/)) 是一个运行在 Kubernetes 上的开源 FaaS 框架。
- 它使用 Docker 容器来交付 FaaS 功能。 
- 函数可以通过不同的触发器（定时器、HTTP 请求、发布消息）激活。
- Fission 允许通过 CLI 命令行或声明式应用程序管理（以 YAML 表示）进行部署管理。
- 可以使用 Argo Workflows ([https://argoproj.github.io/workflows/](https://argoproj.github.io/workflows/)) 进行函数组合扩展。
    

### Why Fission？
![[Pasted image 20260501201155.png]]
- **开源**：可以阅读源码了解细节，参与文档或代码库的改进。
    
- **Kubernetes 原生**：直接运行在 K8s 上。
    
- **相对简单**：相比广泛使用的 Knative FaaS 框架更易上手。
    
- **部署快速简单**：默认情况下，开发者**不需要**构建 Docker 镜像，也不需要使用 Docker 注册表（Registry）。（这是相比 Knative 的巨大优势，Knative 通常需要完成镜像构建和推送流程）。
    
- **支持消息队列**。
    
- **多语言支持**：支持 Python, Node.js, Go, C#, PHP, TensorFlow。
    

## Fission Concept
![[Pasted image 20260501201309.png]]

- **函数 (Function)**：一个软件模块，接收输入、返回结果，可以被触发器独立调用（运行在 Docker 容器中）。
    
- **环境 (Environment)**：函数运行的基础 Docker 镜像。它是语言特定的，包含一个 HTTP 服务器和一些基础库。可以通过“包”进行定制。
    
- **包 (Package)**：用于定制环境的一组文件（通常是源代码和/或编译后的二进制文件）。
    
- **触发器 (Trigger)**：导致函数执行的事件。
    
- **路由器 (Router)**：将 HTTP 请求定向到对应函数的组件。
    
- **规范 (Specifications/Specs)**：定义 Fission 应用程序组件的一组 YAML 文件。相比命令行，它们使部署和维护更简单。
    

### Environments, Packages, and Functions
![[Pasted image 20260501201402.png]]
- 函数运行在 Pod 内的 Docker 容器中，容器基于 Docker 镜像。
- Fission 中的 **Environment** 就是这个基础 Docker 镜像。
- **Package** 是在部署时附加到函数上的定制内容。
    
- **示例**
    - Python 是一个 **Environment**。
    - 为了给 Python 环境添加库（如 ElasticSearch 客户端），需要创建一个 **Package** 并将其添加到函数中，从而在函数创建期间丰富（Enrich）环境。
        

---

## Function Executors

Fission 以两种不同的方式执行函数：
![[Pasted image 20260501201449.png]]
### 1. PoolManager

这是“提前备好容器，按需注入代码”。

- **底层动作**：Fission 会在 K8s 中提前拉起一批**空载**的 Pod（例如纯净的 Python 环境镜像，不含任何业务逻辑）。
    
- **触发瞬间**：当请求到达时，Fission 从资源池中抓取一个空载 Pod，瞬间将你的业务代码（Function + Package）动态注入（Inject）进去并执行。
    
- **生命周期**：执行完毕后，容器会被清理并放回池中待命。
    
- **核心特征**：延迟极低（因为绕过了 K8s 创建 Pod 的耗时步骤），但应对单一函数的瞬间超高并发能力较弱。
    
### NewDeploy

这也是你理解的“按需从头创建”。

- **底层动作**：平时不保持多余的空载容器（或者只保持最低限度）。当流量激增时，Fission 会利用 K8s 的 HPA（水平自动扩缩容）机制，向 K8s 内核下发指令，直接从头创建包含特定业务代码的完整 Pod。
    
- **触发瞬间**：首次请求会触发 K8s 走完整的镜像拉取、网络分配、容器启动流程，将代码和环境打包好后启动。
    
- **核心特征**：首次调用延迟高（冷启动），但在面对持续的重负载时，只要底层物理资源足够，它可以无限横向扩展出成千上万个该函数的专属 Pod。



### 执行器特性对比与参数详解
![[Pasted image 20260501201657.png]]
#### 核心并发参数调节
- **`--requestsperpod` (单 Pod 并发请求数)**
    - **默认值**：1
    - **底层作用**：允许同一个函数实例（Pod）同时并行处理请求的数量
        
- **`--concurrency` (最大并发 Pod 数量)**
    
    - **底层作用**：设置单个函数能够同时运行的 Pod 实例的绝对上限
	- 比如 Fission 允许你设置 `--concurrency=5`，这意味着这个函数最多可以从池子里“抢占” 5 个 Pod 跑自己的实例


| 核心维度       | PoolManager (资源池模式)                                 | NewDeploy (动态部署模式)                                              |
| ---------- | --------------------------------------------------- | --------------------------------------------------------------- |
| **启动延迟**   | **极低 (热启动 Warm Start)**。直接从资源池提取现成的空载 Pod 注入代码即可运行。 | **较高 (默认冷启动 Cold Start)**。请求到达时需从头走完 Kubernetes 创建新 Pod 的完整流程。  |
| **负载应对能力** | **较弱**。无法很好地应对单个函数的极重负载，因为受限于预先设定好的资源池 Pod 数量总上限。   | **极强**。允许同一个函数的多个实例始终处于“开启”状态，可根据 CPU 消耗等核心指标自动拉起海量新 Pod 应对重负载。 |
| **系统默认设置** | Fission 的**默认执行器**。                                 | 需要在部署时手动指定。                                                     |

**Always on**：为了兼顾弹性与响应速度，可以将 `NewDeploy` 执行器配置为**至少保持一个函数 Pod 始终处于“开启”状态 (Always on)**。这使得每个函数都有一个已经预热好的实例并支持动态扩展。

## Autoscaling
![[Pasted image 20260501201929.png]]
- PoolManager 的扩缩容可以使用 Fission 自己的自动扩缩容选项进行定制。
    
- NewDeploy 执行器使用 Kubernetes 核心组件：**HPA (Horizontal Pod Autoscaler)**。当超过设定的目标阈值（例如 Pod 使用超过给定的内存量）时，HPA 会派生（Spawn）新的 Pod。
    
- HPA 也可以在 Fission 外部用于自动缩放 Pod。
    
- **Note**：自动扩缩容对**无状态 (Stateless)** 的 Pod 效果最好。更改有状态 (Stateful) 或非无副作用的 Pod 的数量是非常危险的，（比如 ElasticSearch 数据库节点，它是运行在k8s中地stateful pod， 并依靠固定数量的 Pod 来维持集群共识和数据完整的， 如果用普通的自动扩缩容（HPA）去管理 ElasticSearch。当负载一降，K8s 毫不留情地随机杀掉 2 个 ES Pod，ES 集群就会瞬间丢失数据分片）
    

---

## Fission 命令行界面 (CLI)
![[Pasted image 20260501204153.png]]
- Fission 有一个客户端 CLI，用于与集群交互。
    
- 其风格是 **对象-动作 (Object-Action)**（例如 `fission function list`），这与 OpenStack CLI 类似，但与 `kubectl` 的动作-对象（如 `kubectl get nodes`）不同。
    
|**对象全称**|**缩写**|
|---|---|
|Function|**fn**|
|Environment|**env**|
|Route|**rt**|
|Trigger|**tr**|
|Package|**pkg**|
|Specifications|**spec/specs**|

### 常用 CLI 命令
- 列出环境：`fission environment list`
	
- 列出包：`fission packages list`
	
- 查看函数日志：
```Bash
fission function log --name <function_name> --namespace <namespace>
```
	
- 调用（测试）函数：
```Bash
fission function test --name <function_name> --namespace <namespace>
```
	
- 根据 Specs 规范更新集群：
```Bash
fission specs apply
```

---

## 在 Fission 中编写函数
![[Pasted image 20260501204426.png]]
- 制作简单函数非常容易：
	
1. 在一个名为 `hello.py` 的文件中编写 Python 脚本：
```Python
def main():
    return 'Hello, world!'
```

2.  创建函数：
```Bash
fission function create --name hello --env python --code hello.py
```

3.  测试函数：
```Bash
fission function test --name hello
```

- 上述命令未指定命名空间（Namespace），因此默认为 Kubernetes 的 `default` 命名空间。
- 更新源代码后，必须执行 `fission function update` 命令才能反映更改。
- Docker 容器也可以通过 `fission function run-container` 命令成为 Fission 函数（目前仍处于 Alpha 阶段）。
- 函数可以使用 `fission function list` 列出，使用 `fission function delete` 删除。

---

## 在 Fission 中调用函数
![[Pasted image 20260501204737.png]]
- 虽然 `test` 命令对于快速测试很有用，但它不是正式调用函数的方式。正式调用需要一个**触发器 (Trigger)**。
    
- 最典型的用例是 HTTP 请求，因此需要创建一个**路由 (Route)**（即 HTTP 触发器）：
```Bash
fission route create --name hellort --function hello --method GET --url /hello
```

- 建立端口转发后，我们就可以从集群外部调用该函数：
```Bash
curl http://localhost:9090/hello
Hello world!
```

- 在底层，Fission 使用 **Kubernetes Ingresses** 来实现路由。
- 函数也可以通过其他触发器类型（消息队列、定时器等）调用。
    
---

## Fission 中的 ReSTful API
![[Pasted image 20260501205105.png]]
- 通过添加路由器，可以创建整个 ReSTful API，包括路径参数（例如 ”/users/{:userid}” 中的 userid）。
    
- 我们将 hello.py 路由中的路径参数名称定义为“word”，并限制其为纯字母：
	
```Bash
fission route update --name hello --function hello --method GET --url '/hello/{word:[A-Za-z]+}'
```
	
- 更改 `hello.py` 函数，使其打印带有参数的请求头：
	
```Python
from flask import request

def main():
    return f'Hello, {request.headers["X-Fission-Params-Word"]}!'
```

- 现在，可以通过任何请求更改问候语：
    
```Bash
curl http://localhost:9090/hello/Luca
Hello, Luca!
```

- 可以使用 Flask 的标准对象访问请求参数和请求体，路径参数可以通过 Headers 访问（如上所示）。
    
- 同一个函数可以是多个路由的目标：
    
```Bash
fission route create --name greetingpost --function hello --method POST --url '/greeting/{word:[A-Za-z]+}'
fission route create --name hellodelete --function hello --method DELETE --url '/hello/{word:[A-Za-z]+}'
```

```Bash
curl -XPOST http://localhost:9090/greeting/Luca --data '{}'
Hello, Luca!
curl -XDELETE http://localhost:9090/hello/Luca
Hello, Luca!
```

这里地  hello函数是一个纯函数，delete or post对于其结果没有任何影响， 这里仅做为展示。


---

## Fission 中的环境深度定制
![[Pasted image 20260501211335.png]]

- 有时必须将额外的库或源代码添加到函数中。
    
- Fission 通过使用包 (Packages) 定制环境来实现这一点。
    
- 例如，定制 Python 环境通常包括编写（除了函数脚本外）：
    - 用于依赖项的 `requirements.txt`
    - 在容器上执行 pip install 命令的 `build.sh`
    - 一个 `__init__.py`（可以为空）
        
- 上述内容必须压缩到一个归档文件中，然后作为包添加到集群中：
    
```Bash
fission package create\
--name mypackage\
--sourcearchive mypackage.zip\
--env python\
--buildcmd './build.sh'
```

- 最后一步是使用该包创建函数（注意入口点 entrypoint）：
    
```Bash
fission fn create --name myfunction\
--env python\
--pkg mypackage\
--entrypoint "myfunction.main"
```

### Fission 归档文件类型 (Archives)
![[Pasted image 20260501212243.png]]
- Fission 有两种类型的归档文件：
	- **源归档文件 (`--sourcearchive` 选项)**：是指 Fission 在解压后尝试编译并安装依赖项的文件。
	- **部署归档文件 (`--deployarchive` 选项)**：是指 Fission 仅仅进行解压的文件。
    
- 当需要将可执行代码或数据文件添加到函数中时，部署归档文件非常有用。
    

## Other Triggers

![[Pasted image 20260501213509.png]]
### Timers
- 可以使用定时器定期调用 Fission 函数：
    
```Bash
fission timer create\
--name everyminuteMyFunction\
--function myfunction\
--cron "@every 1m"
```

- **注意**：定时器间隔必须长于函数的运行时间，以避免运行中的函数数量不断增加（导致系统过载）。

### Kubernetes Watches
- 当 Kubernetes 资源（Pods, Ingresses 等）发生变化时，也可以调用 Fission 函数：
	
```Bash
fission watch create\
--name podEventsMyFunction\
--function myfunction\
--type pod
```

---

## 异步架构模式

### 1. Fission 消息队列
![[Pasted image 20260501215043.png]]
- Fission 可以与消息队列集成，以便函数可以存储消息，并在稍后由另一个函数消费。
    
- 通过这种方式可以实现**异步函数**（例如，一个函数可以将消息存储在队列中，供另一个函数在以后提取）。
    
- 队列必须由持久化机制提供支持，例如 **Apache Kafka** 或 **Redis**（它们是快速存储系统，能够处理高负载并保持良好的性能）。
    

### 2. Fission Websockets 
![[Pasted image 20260501215421.png]]
- 运行时间长的任务最好由异步函数处理，但这与 HTTP 请求的同步特性不太相符（会导致客户端请求超时）
    
- 假设我们有一个用于启动机器学习任务（可能需要几分钟）的 JavaScript 客户端；一个可能的解决方案如下：
    
    1. 客户端发送 HTTP POST 请求。
    2. Fission 将任务请求存储到队列中，并返回 HTTP 状态 202（表示请求已接受）。
    3. 客户端打开一个 Websocket（这是客户端和服务器之间的一个双向、非阻塞的通道）。
    4. 一个 Fission 函数从队列中获取请求并执行它。
    5. 一旦函数完成其任务，Websocket 通道将用于将结果发送回客户端。
        
- 通过这种方式，Fission 可以在后端应用程序和 Web 前端之间容纳长时间运行的任务。 
- Node.js 环境中提供了 Websockets。

---

## Fission 规范与声明式管理 (Specifications)

![[Pasted image 20260501221038.png]]
- 使用 Fission CLI 效率不高，因为开发人员在每次更改组件（函数、路由等）时都必须发出命令 发出命令违背了两个关键原则：基础设施即代码 (Infrastructure as Code) 和声明式应用程序管理 (Declarative Application Management)。
    
- 解决方案是**使用规范（或称为 specs）：目录下的一组 YAML 文件，可以通过 `fission specs apply` 一次性应用到集群上（默认情况下目录名为 `specs`）。**

- 您可以使用 `fission specs apply --delete` 将 specs 目录中的资源与集群上的资源同步（这会从集群中**删除**不在 specs 中的资源）。
    
- 命令 `fission specs destroy` 会从集群中消除 specs 目录中的所有资源。
    

## 向 Fission 函数传递参数 (ConfigMaps)

![[Pasted image 20260501221114.png]]
- 在集群中传递参数的一个便捷方法是使用 Kubernetes ConfigMaps。
- Kubernetes 使用此功能在 Pod 内设置环境变量或文件，而无需显式传递它们的值：
    
```YAML
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: default
  name: shared-data
data:
  ES_USERNAME: elastic
  ES_PASSWORD: elastic
```

```Bash
kubectl apply -f shared-data.yaml
```

- （当然，明文使用 Credentials 不是好主意，Kubernetes 为此提供了 Secrets 功能，但此例仅作示例）
    
- 以下是在 Python 中使用它们的方法（变量的值作为文件写入与 ConfigMap 同名的命名空间和目录中）：
    
```Python
def config(k):
    with open(f'/configs/default/shared-data/{k}', 'r') as f:
        return f.read()
```

- Fission 使用文件是因为 ConfigMaps 的值可以容纳复杂类型（比如 JSON 对象），不仅仅限于数字和字符串。
    

## Fission 函数超时设置
![[Pasted image 20260501221209.png]]
- 创建函数时，可以设置不同类型的 Fission 超时选项。
- **`--specializationtimeout`**：函数创建（专业化）超时（默认 120 秒）。
- **`--fntimeout`**：函数执行超时（默认 60 秒）。
- **`--idletimeout`**：函数回收前的空闲超时（默认 120 秒）。
- 其他超时限制（如入口控制器 Ingress Controller、代理 Proxies 等）也可能适用。

---

# Part 5: Testing FaaS 

## 测试基础理论回顾

### Why We Test? 
![[Pasted image 20260502110514.png]]

### How We Test? 
![[Pasted image 20260502110541.png]]

### What We Test? 
![[Pasted image 20260502110621.png]]

### Test Taxonomy 
![[Pasted image 20260502110646.png]]
- **单元测试 (Unit tests)**：隔离测试各个独立的模块。
    
- **集成测试 (Integration tests)**：测试软件与其他组件（例如数据库）的集成情况。为了测试错误条件，可以使用 **Mock（模拟对象）**。
    
    > **辅助理解**：Mock 就像是拍电影时的“替身演员”。它是一个用来模拟真实组件（如数据库）的软件片段，有了它，开发人员就可以随意触发并模拟各种极端错误条件（比如强制让 Mock 数据库断开连接），而不用真的去破坏生产环境的数据库。
    
- **系统测试 / 端到端测试 (System tests / end-to-end tests)**：测试整个软件作为一个整体的行为（完全站在客户端的视角来看）。
    

---

## FaaS 测试实战：四次迭代开发 (Toy Application)

本节将通过一个带有 ReSTful API 的小型 FaaS 应用程序及其测试代码的演进，模拟开发团队的四次迭代过程。应用需求包括对 Student 和 Course 数据的增删改查 (CRUD) 操作。
![[Pasted image 20260502110741.png]]

### 架构与 API 设计基础

- **技术栈**：Python, Fission, ElasticSearch。
    
- **环境初始化**：应用程序依赖于特定的 Python 环境，通过以下命令创建：
    
```Bash
fission env create --spec --name python --image fission/python-env --builder fission/python-builder
```

![[Pasted image 20260502111328.png]]
- **API 路由空间**：
    
    - `/students` (GET)
    - `/students/{studentid:}` (GET, PUT, DELETE)
    - `/courses` (GET)
    - `/courses/{courseid:}` (GET, PUT, DELETE)
    - `/courses/{courseid:}/students` (GET)
        

---

### 第一轮迭代：占位符函数与 E2E 骨架 (First Iteration)

此阶段的目标是打通框架，先写出没有真实业务逻辑的“占位符 (Dummy)”函数，并建立最基础的端到端测试来验证路由是否通畅。

![[Pasted image 20260502111409.png]]

- **编写 Dummy 函数** (`./functions/course/course.py`)：
    
```Python
from flask import request, current_app
import requests, logging, json

def main():
    current_app.logger.debug(f'Received request: {request.method}')
    return 'course', 200
```

- **使用 Specs 创建函数和路由**：
    ![[Pasted image 20260502112118.png]]
```Bash
fission function create --name course --spec --env python --code ./functions/course/course.py
fission route create --spec --name courseget --function course --url '/courses/{courseid:[0-9]+}' --method GET
fission specs apply
```

- 带上 `--spec` 后，Fission **不会**立刻去操作真实的集群。相反，它会把你的意图翻译成 Kubernetes 能看懂的 YAML 格式，并把这个 YAML 文件悄悄保存在你本地的一个文件夹（默认叫 `specs`）里。
- 当你执行 `fission specs apply` 时，Fission 会拿着你本地目录specs里的这厚厚一沓“YAML 蓝图”，跑去跟真实的 Kubernetes 集群对照



- **编写最简单的端到端测试 (E2E Tests)**：
	
![[Pasted image 20260502112229.png]]
	
```Python
class TestEnd2End(unittest.TestCase):
    def test_student(self):
	    # 测试基础设施与网络连通性。router工作并找到路由规则，成功唤醒 Pod，函数没有发生崩溃
        self.assertEqual(test_request.get('/students/111').status_code, 200)
        # 测试路由调用的准确性， 验证了 `studentget` 路由确实精准命中并执行了 `student`
        self.assertEqual(test_request.get('/students/111').text, 'student')
        self.assertEqual(test_request.put('/students/111', {}).status_code, 200)
        self.assertEqual(test_request.put('/students/111', {}).text, 'student')
        self.assertEqual(test_request.delete('/students/111').status_code, 200)
        self.assertEqual(test_request.delete('/students/111').text, 'student')

    def test_students(self):
        self.assertEqual(test_request.get('/students').status_code, 200)
        self.assertEqual(test_request.get('/students').text, 'students')
```

> **辅助理解**：第一轮测试仅仅是为了验证“路由存在且正确绑定到了函数上”，以及“函数能够成功返回状态码 200”。

- **步骤 1：打通网络通道**：必须先执行 `kubectl port-forward` 命令，将 Kubernetes 集群内部路由器 (Router) 的 80 端口映射到本地的 9090 端口。
    
- **步骤 2：执行测试**：在本地运行 Python 测试脚本 (`tests/end2end.py`)，预期结果应为成功。

---

### 第二轮迭代：引入参数与连接数据库 

#### 1. 设置全局参数 (Parameters)

- **使用 ConfigMap**：我们使用 Kubernetes 的 ConfigMap 来集中管理全局参数。
    
![[Pasted image 20260502113512.png]]
	
```YAML
# ---------------------------------------------------------
# specs/configmap-parameters.yaml
# 定义名为 'parameters' 的 ConfigMap，存储数据库连接信息
# ---------------------------------------------------------
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: default
  name: parameters
data:
  # ElasticSearch 集群在 K8s 内部的 DNS 访问地址
  ES_URL: 'https://elasticsearch-master.elastic.svc.cluster.local:9200'
  ES_USERNAME: 'elastic'
  ES_PASSWORD: 'elastic'
  ES_DATABASE: 'students'
```

- **应用 ConfigMap：使用 `kubectl apply -f specs/configmap-parameters.yaml` 命令将该 ConfigMap 应用到集群中。

```YAML
# ---------------------------------------------------------
# 函数的 YAML spec 文件 (如 function-student.yaml) 的片段
# 作用：指示 Fission 在启动函数 Pod 时，将名为 'parameters' 的 ConfigMap 挂载进去
# ---------------------------------------------------------
configmaps:
  - name: parameters
    namespace: default
```

#### 2. 初始化数据库 (Database)

- **端口转发与创建数据库**：在进行了 ElasticSearch 端口转发之后，我们在 ElasticSearch 中创建一个包含 students（学生）和 courses（课程）数据的数据库（索引）。
    
![[Pasted image 20260502113718.png]]
	
```Bash
# 将 K8s 内部的 ElasticSearch 9200 端口映射到本地的 9200 端口
kubectl port-forward service/elasticsearch-master -n elastic 9200:9200

# 使用 curl 发送 PUT 请求，创建一个名为 'students' 的索引 (数据库)，并定义其结构
curl -XPUT -k 'https://127.0.0.1:9200/students' \
  --user 'elastic:elastic' \
  --header 'Content-Type: application/json' \
  --data '{
  "settings": {
    "index": {
      "number_of_shards": 3,   # 数据分片数为 3，用于分布式存储和提高检索性能
      "number_of_replicas": 1  # 副本数为 1，提供高可用性和容错能力
    }
  },
  "mappings": {              # 定义文档的数据结构 (Schema)
    "properties": {
      "id": {
        "type": "keyword"      # keyword 类型：精确匹配，不进行全文分词
      },
      "type": {
        "type": "keyword"
      },
      "timestamp": {
        "type": "date"         # date 类型：处理时间戳
      },
      "name": {
        "type": "keyword"
      },
      "courses": {
        "type": "keyword"
      }
    }
  }
}'
```

#### 3. 升级函数与测试 (Functions)

- **函数读取数据库：让我们检查 Fission 函数是否能够使用通过 ConfigMap 共享的参数，成功从 ElasticSearch 数据库中读取数据。
	
 ![[Pasted image 20260502113816.png]]
	
```Python
# ---------------------------------------------------------
# 升级后的函数代码 (如 students.py)
# 作用：从 ConfigMap 挂载的文件中读取配置，并向 ElasticSearch 发起真实的请求
# ---------------------------------------------------------
from flask import request, current_app
import requests, logging, json

# 辅助函数：读取 ConfigMap 文件
# Fission 会把 ConfigMap 里的每一个键值对，变成 Pod 内部 /configs/... 目录下的一个个文件
def config(k):
    with open(f'/configs/default/parameters/{k}', 'r') as f:
        return f.read()

def main():
    # 使用 requests 库发起真实的 GET 请求，获取刚才创建的 'students' 索引信息
    r = requests.get(f'{config("ES_URL")}/{config("ES_DATABASE")}',
                     verify=False, # 忽略 SSL 证书验证 (用于测试环境)
                     auth=(config("ES_USERNAME"), config("ES_PASSWORD"))) # 注入账密
    
    # 不再返回写死的 'student' 字符串，而是返回真实的 JSON 响应和状态码
    return r.json(), r.status_code
```

- **测试断言升级 (The tests now check...)**：现在的测试代码将检查函数是否真正返回了一个 JSON 对象。
    
```Python
# ---------------------------------------------------------
# 升级后的端到端测试片段
# ---------------------------------------------------------
def test_students(self):
    # 验证 HTTP 连通性依然正常 (200 OK)
    self.assertEqual(test_request.get('/students').status_code, 200)
    
    # 【重大改变】：不再断言返回特定字符串，而是断言返回值是一个非空的 JSON 对象
    # 这证明了函数成功连接了数据库，获取了数据，并序列化为 JSON 传回了客户端
    self.assertIsNotNone(test_request.get('/students').json())
```

- **阶段性成果***：它现在开始像一个真正的应用程序了：它查询数据库并从中返回一些内容，并且我们仍然可以通过端到端测试证明它正常工作……现在是时候利用数据库数据做更多的事情了。

---

### 第三轮迭代：处理副作用与事务
![[Pasted image 20260502121510.png]]
- 到目前为止，被测试的函数都是无副作用的 (side-effect-free)，但这并不是我们想要测试的目标，因为实际需求要求执行事务处理（例如将学生添加到数据库，从数据库中删除课程等）。
    
- 为了测试事务，我们需要一个可以随意擦除并填充数据的数据库：一个测试数据库。
    
- 为了保持简单，我们将使用同一个“students”数据库（这不是一个好习惯，但为了演示是可以接受的）。
    
- 让我们编写一个用于重新创建数据库的函数，以便用于测试目的（同时，这也便于在代码中记录和说明数据库的结构）。
    
- 在此之前，让我们将数据库模式 (schema) 作为一个参数添加到我们的 ConfigMap 中：
    
```YAML
data:
  ES_URL: 'https://elasticsearch-master.elastic.svc.cluster.local:9200'
  ES_USERNAME: 'elastic'
  ES_PASSWORD: 'elastic'
  ES_DATABASE: 'students'
  ES_SCHEMA: |
    {
      "settings": {
        "index": {
          "number_of_shards": 3,
          "number_of_replicas": 1
          # ... (下周我们将在 ElasticSearch 课程中详细讨论分片/副本 shards/replicas)
```

- 必须在集群上更新 ConfigMap：
    
```Bash
kubectl apply -f specs/configmap-parameters.yaml
```

### 第三轮迭代：擦除测试数据库 (Third Iteration - Wipe Test Database)

- 该函数使用我们刚刚创建的 `ES_SCHEMA` 参数来删除并重新创建一个 ElasticSearch 数据库。
    
```Python
def main():
    # 发送 DELETE 请求删除现有数据库
    r = requests.delete(f'{config("ES_URL")}/{config("ES_DATABASE")}',
                        verify=False,
                        auth=(config("ES_USERNAME"), config("ES_PASSWORD")))
    # 发送 PUT 请求，使用 ConfigMap 中的 ES_SCHEMA 重新创建干净的数据库
    r = requests.put(f'{config("ES_URL")}/{config("ES_DATABASE")}',
                     verify=False,
                     auth=(config("ES_USERNAME"), config("ES_PASSWORD")),
                     data=json.loads(config("ES_SCHEMA")))
    return r.json(), r.status_code
```

- 然后，我们必须创建函数和路由，手动将 ConfigMap 添加到刚刚创建的函数 spec 文件中，最后应用这些 specs：
    
```Bash
fission function create --name wipedatabase --spec --env python --code ./functions/wipedatabase/wipedatabase.py

# (需要手动编辑 function-wipedatabase.yaml 文件，将 parameters ConfigMap 挂载进去)

fission route create --spec --name wipedatabase --function wipedatabase --url '/wipedatabase' --method DELETE

fission specs apply
```

- 现在是时候测试 wipedatabase 函数了：
    
```Bash
curl -XDELETE 'http://127.0.0.1:9090/wipedatabase'
```

- 获取到的响应 (Getting)： `{"acknowledged":true,"index":"students","shards_acknowledged":true}`
    

### 第三轮迭代：测试 #1 (Third Iteration - Tests #1)

- 包含在每次测试前擦除数据库的事务测试用例：
    
```Python
    def setUp(self):
        # 每次测试运行前，调用 wipedatabase 路由清空数据，保证测试环境的绝对纯净
        self.assertEqual(test_request.delete('/wipedatabase').status_code, 200)
        # 辅助理解：ElasticSearch 操作需要极短的时间生效，sleep 保证底层刷新完成
        time.sleep(1)

    def test_student(self):
        # 测试新增数据 (PUT 返回 201 Created)
        self.assertEqual(test_request.put('/students/1', {'name': 'John Doe', 
				        'courses': '90024'}).status_code, 201)
        self.assertEqual(test_request.put('/students/2', {'name': 'Jane Doe', 
				        'courses': ['90024', '90059']}).status_code, 201)
        time.sleep(1)

        # 验证写入的数据是否可以通过 GET 正确读取
        r= test_request.get('/students/1')
        self.assertEqual(r.status_code, 200)
        o= r.json()['_source']
        self.assertEqual(o['name'], 'John Doe')
        self.assertEqual(o['courses'], '90024')

        r= test_request.get('/students/2')
        o= r.json()['_source']
        self.assertEqual(r.status_code, 200)
        self.assertEqual(o['name'], 'Jane Doe')
        self.assertEqual(o['courses'], ['90024', '90059'])
```

### 第三轮迭代：测试 #2 (Third Iteration - Tests #2)

```Python
        # 测试覆盖更新 (Update)：将 1 号学生的名字修改为 Bob Carr
        r= test_request.put('/students/1', {'name': 'Bob Carr', 'courses': 
					        '90024'})
        self.assertEqual(r.status_code, 201)

        # 验证修改是否生效
        r= test_request.get('/students/1')
        o= r.json()['_source']
        self.assertEqual(r.status_code, 200)
        self.assertEqual(o['name'], 'Bob Carr')
        self.assertEqual(o['courses'], '90024')

        # 【测试错误条件】：请求不存在的 999 号学生，系统应优雅地返回 404 而不是崩溃
        self.assertEqual(test_request.get('/students/999').status_code, 404)

        # 测试删除事务
        self.assertEqual(test_request.delete('/students/1').status_code, 200)
        self.assertEqual(test_request.delete('/students/2').status_code, 200)
```

- 显然这些测试会失败 (fail)，正如它们应该的那样，因为目前的函数中还没有编写更改数据库的代码。
    
- 在通过更改应用程序代码使其工作之前，让测试用例先失败是一个好习惯（通过这种方式，您可以确定该测试在成功或失败这两种情况下都能正常运行）。
    
- 您可能会注意到，有一个测试用例是专门用于测试错误条件 (error conditions) 的（即测试 404）
    

### 第三轮迭代：实现事务 (Third Iteration - Implement Transactions)

- 更改数据库的 `student` 函数：
    
```Python
# 辅助函数：利用 Fission 解析并传递在 Headers 中的参数，动态拼接具体文档的 URL
def docurl():
    return f'{config("ES_URL")}/{config("ES_DATABASE")}/_doc/student_{request.headers["X-Fission-Params-Studentid"]}'

# 辅助函数：组装数据，补充时间戳和 ID
def addfields(data):
    data['timestamp'] = datetime.now().isoformat()
    data['id'] = request.headers['X-Fission-Params-Studentid']
    data['type'] = 'student'
    return data

def main():
    try:
        if request.method == 'GET':
            r = requests.get(docurl(), verify=False, auth=(config("ES_USERNAME"), 
				            config("ES_PASSWORD")))
            return r.json(), r.status_code
            
        elif request.method == 'PUT':
            # 并发控制 (乐观锁)：修改前先获取当前文档的 _seq_no 和 _primary_term
            r = requests.get(docurl(), verify=False, auth=(config("ES_USERNAME"), 
					        config("ES_PASSWORD")))
            if r.status_code == 200:
                params={'if_seq_no': r.json()['_seq_no'], 'if_primary_term': 
			                r.json()['_primary_term']}
            else:
                params= {}
                
            # 执行写入：携带并发控制参数，如果在这期间数据被其他进程修改，写入将被 ES 拒绝
            r = requests.put(docurl(), verify=False, auth=(config("ES_USERNAME"), 
					            config("ES_PASSWORD")),
                             headers={'Content-type': 'application/json'},
                             data=json.dumps(addfields(request.json)),
                             params=params)
            return r.json(), r.status_code
            
        elif request.method == 'DELETE':
            # 并发控制：删除前同样需要获取版本号
            r = requests.get(docurl(), verify=False, auth=(config("ES_USERNAME"), 
					            config("ES_PASSWORD")))
            if r.status_code != 200:
                return r.json(), r.status_code
                
            r = requests.delete(docurl(), verify=False, auth=
						            (config("ES_USERNAME"), config("ES_PASSWORD")),
                                params={'if_seq_no': r.json()['_seq_no'], 
	                            'if_primary_term': r.json()['_primary_term']})
	    return r.json(), r.status_code
            
    except Exception as e:
        current_app.logger.error(e)
        return {'message': f'Error {e}'}, 500
        
    return {'message':'Method not allowed'}, 405
```

### 第三轮迭代：查询 (Third Iteration - Queries)

```Python
def main():
    try:
        # 向 ES 的 _search 端点发送 POST 请求执行复杂查询
        r = requests.post(f'{config("ES_URL")}/{config("ES_DATABASE")}/_search',
                          verify=False,
                          auth=(config('ES_USERNAME'), config('ES_PASSWORD')),
                          headers={'Content-type': 'application/json'},
                          data=json.dumps({
                              '_source': False, # 关闭完整的原始 JSON 返回
                              'query': {
                                  'term': {
                                      'type': {
	                                      # 仅查询类型为 student 的数据
                                          'value': 'student' 
                                      }
                                  }
                              },
                              'fields':[ # 明确指定需要返回的字段列表
                                  {'field':'id'},
                                  {'field':'timestamp'},
                                  {'field':'name'},
                                  {'field':'courses'}
                              ],
                              'sort': [ # 定义排序规则
                                  {
                                      'name': {
                                          'order': 'asc', # 按 name 升序排列
                                          'missing': '_last',
                                          'unmapped_type': 'keyword'
                                      }
                                  }
                              ]
                          })
                          )
        return r.json(), r.status_code
    except Exception as e:
        current_app.logger.error(e)
        return {'message': f'Error {e}'}, 500
        
    return {'message':'Method not allowed'}, 405
```

### 第三轮迭代：测试查询 (Third Iteration - Test Queries)

```Python
def test_students(self):
    # 准备测试数据：向数据库写入两门课程和两个学生
	self.assertEqual(test_request.put('/courses/90024', {'name': 'Cloud 
										Computing'}).status_code, 201)
    self.assertEqual(test_request.put('/courses/90059', {'name': 'Introduction to 
									    Programming'}).status_code, 201)
    self.assertEqual(test_request.put('/students/1', {'name': 'John Doe', 
									    'courses': '90024'}).status_code, 201)
    self.assertEqual(test_request.put('/students/2', {'name': 'Jane Doe', 
						    'courses': ['90024', '90059']}).status_code, 201)
        
    # 辅助理解：ES 是近实时搜索引擎，写入后需等待 1 秒钟以便底层索引刷新，否则后续查询将返回空结果
    time.sleep(1)
        
    # 执行集合查询请求
    r= test_request.get('/students')
    o= (r.json()['hits'])['hits']
    self.assertEqual(r.status_code, 200)
        
    # 验证排序规则：'Jane' 的字母顺序先于 'John'，因此 Jane 必须位于结果集的第一位 (索引 0)
    self.assertEqual(o[0]['fields']['name'][0], 'Jane Doe')
    self.assertEqual(o[1]['fields']['name'][0], 'John Doe')
```

---

### 第四轮迭代：重构 #1 (Fourth Iteration - Refactoring #1)

- 由于端到端测试运行正常，这说明应用程序正在正常工作……但是代码库显得非常臃肿，存在大量重复代码，并且普遍缺乏对公共行为的抽象。
    
- 解决这个问题的答案是重构 (Refactoring)，即在保持代码当前行为不变的前提下，重新组织代码结构。
    
- 例如，我们可以使用 4 个类来重构软件（参考 UML 类图）：`ESStudent`, `ESDocument`, `ESCourse`, `ESCommons`。
![[Pasted image 20260502123717.png]]

### 第四轮迭代：重构 #2 (Fourth Iteration - Refactoring #2)

- 一些公共行为被分组到 `Commons` 类中：
    
```Python
# 将配置读取、鉴权信息生成和通用 URL 拼接提取为静态方法
class Commons:
    # 扩容解析：使用 @staticmethod 装饰器，意味着这些方法不需要实例化对象即可直接调用
    # 这在 FaaS 环境中非常轻量，充当了全局辅助工具箱的角色
    @staticmethod
    def config(k):
        with open(f'/configs/default/parameters/{k}', 'r') as f:
            return f.read()

    @staticmethod
    def auth():
        return (Commons.config("ES_USERNAME"), Commons.config("ES_PASSWORD"))

    @staticmethod
    def search_url():
        return 
	        f'{Commons.config("ES_URL")}/{Commons.config("ES_DATABASE")}/_search'
```

### 第四轮迭代：重构 #3 (Fourth Iteration - Refactoring #3)

- 另一个类可以持有学生和课程所共有的行为……这就是 `ESDocument` 类：
    
```Python
class ESDocument:
    # 初始化方法，接收公共组件、请求对象和文档类型 (student 或 course)
    def __init__(self, commons, req, type):
        self.commons = commons
        self.req = req
        self.type = type

    # 动态拼接 ElasticSearch 的绝对 URL
    def url(self):
	    return f'{self.commons.config("ES_URL")}/{self.commons.config("ES_DATABASE")}/_doc/{self.type}_{self.req.headers[f"X-Fission-Params-{self.type.capitalize()}id"]}'

    # 统一注入系统级字段 (如时间戳和ID)
    def add_fields(self, data):
        data['timestamp'] = datetime.now().isoformat()
        data['id'] = self.req.headers[f'X-Fission-Params-
								        {self.type.capitalize()}id']
        data['type'] = self.type
        return data

    # 封装 GET 方法逻辑
    # 扩容解析：原课件代码中存在微小的命名不一致，上方定义了 url(self)，下方调用了 
	# self.doc_url()。在真实执行时需统一命名。
    def get(self):
        r = requests.get(self.doc_url(), verify=False, auth=self.commons.auth())
        return r.json(), r.status_code

    # 封装 PUT 方法逻辑 (包含乐观锁并发控制)
    def put(self):
        r = requests.get(self.doc_url(), verify=False, auth=self.commons.auth())
        if r.status_code == 200:
            params={'if_seq_no': r.json()['_seq_no'], 'if_primary_term': r.json()
		            ['_primary_term']}
        else:
            params= {}
        r = requests.put(self.doc_url(), verify=False, auth=self.commons.auth(),
                         headers={'Content-type': 'application/json'},
                         data=json.dumps(self.add_fields(request.json)),
                         params=params)
        return r.json(), r.status_code

    # 封装 DELETE 方法逻辑
    def delete(self):
        r = requests.get(self.doc_url(), verify=False, auth=self.commons.auth())
        # ... 
```

### 第四轮迭代：重构 #4 (Fourth Iteration - Refactoring #4)

- The classes for course and document are just a specialization of the base class：
    
```Python
from ESDocument import ESDocument

# 扩容解析：面向对象的继承在 FaaS 中依然适用。
# ESCourse 继承了 ESDocument 的所有 CRUD 方法，只需在初始化时指定 type 为 'course' 即可。
class ESCourse(ESDocument):
    def __init__(self, commons, req):
        super(ESCourse, self).__init__(commons, req, 'course')
```

- 现在的 `wipedatabase` 函数可以被精简为类似下面的样子：
    
```Python
from Commons import Commons

def main():
    # 直接调用 Commons 类的静态方法，大幅减少了原有的冗余代码
    r = requests.delete(f'{Commons.config("ES_URL")}/{Commons.config("ES_DATABASE")}',
                        verify=False,
                        auth=Commons.auth())
    r = requests.put(f'{Commons.config("ES_URL")}/{Commons.config("ES_DATABASE")}',
                     verify=False,
                     auth=Commons.auth(),
                     headers={'Content-type': 'application/json'},
                     data=Commons.config("ES_SCHEMA"))
    return r.json(), r.status_code
```

- 如此激进的重构将会是充满风险的……但是我们编写的端到端测试向我们保证，应用程序依然在按照预期工作。
    

### 第四轮迭代：重构 #5 (Fourth Iteration - Refactoring #5)

- 将源代码依赖项添加到 Fission 函数中，需要对 specs 目录下的函数定义文件进行更改：
    
    > **扩容解析**：重构后，各个功能被拆分到了不同的 Python 文件中。为了让 Fission 在执行时能找到这些公共类，必须利用 `ArchiveUploadSpec` 将多个相关文件打包成一个 Package。
    
```YAML
# 在 YAML 的 ArchiveUploadSpec 中显式声明需要打包包含的文件
include:
- ./functions/wipedatabase/wipedatabase.py
- ./functions/library/Commons.py
kind: ArchiveUploadSpec
name: functions-wipedatabase-wipedatabase-py-1WkK
---
apiVersion: fission.io/v1
kind: Package
metadata:
  creationTimestamp: null
  name: wipedatabase-1a9262b4-002e-48ce-bdb0-cb5400ea777d
# ...
---
apiVersion: fission.io/v1
kind: Function
metadata:
  creationTimestamp: null
  name: wipedatabase
spec:
# ...
  package:
    functionName: wipedatabase.main
    # 绑定上方定义好的包含依赖文件的 Package
    packageref:
      name: wipedatabase-1a9262b4-002e-48ce-bdb0-cb5400ea777d
      namespace: default
  requestsPerPod: 1
  resources: {}
```

### 第四轮迭代：单元测试 (Fourth Iteration - Unit Tests)

- 添加一些单元测试永远都不嫌晚：
    
```Python
class TestUnit(unittest.TestCase):
    def setUp(self):
        # 扩容解析：单元测试的核心是“隔离”。
        # 使用 MagicMock 模拟 Commons 和 request 对象，这样测试运行时根本不会产生真实的外部网络请求。
        self.mock_commons = MagicMock(autospec=Commons)
        self.mock_commons.config = MagicMock(return_value='x')
        self.mock_request = MagicMock(autospec=request)
        self.mock_request.headers ={'X-Fission-Params-Courseid': 'comp90024',
                                    'X-Fission-Params-Studentid': '123'}

    def test_escourse(self):
        # 验证 ESCourse 类的 URL 拼接逻辑是否正确
        test_escourse= ESCourse(self.mock_commons, self.mock_request)
        self.assertEqual(test_escourse.url(), 'x/x/_doc/course_comp90024')

    def test_esstudent(self):
        # 验证 ESStudent 类的 URL 拼接逻辑是否正确
        test_esstudent= ESStudent(self.mock_commons, self.mock_request)
        self.assertEqual(test_esstudent.url(), 'x/x/_doc/student_123')
```

---

## In Closing

![[Pasted image 20260502124407.png]]
    

### 绝对不能做的事情，但为了演示我还是做了 (What Must Not Be Done (But I Did Anyway))
![[Pasted image 20260502124420.png]]
- **测试数据库必须与其他数据库分开独立运行**。（扩容：绝对不能像课件里那样在主数据库里建测试表，物理隔离是底线）。
    
- **凭证必须存储在 Kubernetes Secrets 中，而不是 ConfigMaps 中**。（扩容：ConfigMap 是明文存储，存在严重的安全泄露风险）。
    
- **必须开发集成测试**，以测试那些无法轻易复现的条件（例如 DBMS 变得不可访问、DBMS 返回格式错误的 JSON 等）。这意味着需要实现一个模拟的 DBMS 端点，但这超出了本次讲座的范围。
    
- **数据库模式 (映射 mappings) 并不理想**。例如，对课程和学生使用了相同的模式……不过话又说回来，ElasticSearch 本来就不是处理严密事务 (transactions) 的理想选择。
    
- **用于测试的 `/wipedatabase` 路由绝对不能对外公开访问**。（扩容：这是极其危险的后门，生产环境中这会导致“删库跑路”级别的灾难）。
    
- **应用程序不应该将 ElasticSearch 的原始响应原封不动地返回给客户端**。（扩容：这暴露了底层基础设施的数据结构，应在 FaaS 函数中封装一层标准化的数据传输对象 DTO）。
    
- **应用程序应该对传入的 JSON 对象的模式 (schema) 进行严格校验**。（扩容：永远不要信任客户端传来的数据输入）。
    
- **应用程序没有对搜索结果进行分页 (paginate) 处理**。（扩容：如果数据量庞大，这会导致严重的性能瓶颈和内存溢出）。
    
- **该应用程序仅作为一个教学工具，绝对不要将其部署到生产环境中**