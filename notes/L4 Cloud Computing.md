# Cloud Cost

![[Pasted image 20260327092813.png]]

![[Pasted image 20260327093131.png]]

egress fees: The money to take your data out from the cloud service provider. They wanna u to keep data in and continuous use their service instead of take it out.


# Def
![[Pasted image 20260327093500.png]]

# The Most Common Cloud Models
![[Pasted image 20260327093510.png]]
- **SaaS (Software as a Service)**  给你**成品软件**
    直接提供软件给用户使用。  
    用户通常只管“用”，不用管底层平台和基础设施。  
    例子：Gmail、Office 365、Salesforce
- **PaaS (Platform as a Service)**  给你**开发/部署平台**
    提供开发和运行应用的平台。  
    用户主要关心代码和应用逻辑，不需要自己管理底层 OS、runtime、部分中间件。  
    例子：Heroku、Google App Engine
- **IaaS (Infrastructure as a Service)**  给你**基础机器资源**
    提供最基础的计算资源，比如虚拟机、存储、网络。  
    用户自己管理更多内容，比如 OS、软件环境、应用。  
    例子：AWS EC2、Azure VM


![[Pasted image 20260327093755.png]]
**Public**：公有云  
面向公众或大量客户开放，由云服务商提供，比如 `AWS、Azure、GCP`。

![[Pasted image 20260327093818.png]]
Private：私有云  
资源给单个组织自己使用，控制力强，常见于企业内部或高安全需求场景

![[Pasted image 20260327093831.png]]
**Hybrid**：混合云  
把两种或多种云结合起来使用，常见是“私有云 + 公有云”，兼顾弹性和控制力。


# `*aaS` Delivery Models
![[Pasted image 20260327094916.png]]

**Main focus of this course: Infrastructure As A Service (IaaS)**


# Melbourne Research Cloud and NeCTAR
![[Pasted image 20260327100149.png]]
![[Pasted image 20260327100216.png]]
MRC仅对unimelb开放，zone很小。

![[Pasted image 20260327100506.png]]
NeCTAR可以免费使用
https://tutorials.rc.nectar.org.au/allocation-management/03-account-and-trial

**VOLUME in NeCTAR cannot be shrink!!! BECAREFUL with extend. It can only go up.**

---

# OpenStack

OpenStack 是一套用于把计算、存储和网络资源统一管理并按需提供的开源 IaaS 云平台。

![[Pasted image 20260327101355.png]]

用户通常是：
- 先通过 **Dashboard**
- 由 **Identity** 做认证授权
- 再去调用 **Compute / Network / Image / Block Storage / Object Storage** 这些实际资源服务

![[Pasted image 20260327102219.png]]

---
## Identity
 ![[Pasted image 20260328122243.png]]

---

## Horizon
 ![[Pasted image 20260328122322.png]]
Horizon = OpenStack 的 Web 控制台 / 图形化管理界面

---

## Nova
![[Pasted image 20260328122444.png]]
Nova 负责管理 **compute instance（计算实例，通常就是 VM）** 的整个生命周期，例如：
- 创建（spawn）
- 启动 / 停止
- 调度到某台物理机
- 删除 / 回收（decommission）

本质上就是 OpenStack 的 **计算资源管理器**。

![[Pasted image 20260328122508.png]]
- **nova-api**：接收并响应用户的 VM 管理请求，是 Nova 的 API 入口。
- **nova-compute**：在计算节点上调用底层虚拟化器，真正创建、运行和删除 VM。
- **nova-scheduler**：根据资源和策略选择哪台宿主机来运行新的 VM。
- **nova-conductor**：负责在 compute 服务与数据库等其他组件之间做协调和中介。
- **nova-network**：负责处理网络相关任务，但在现代 OpenStack 中大多已被 Neutron 取代。


![[Pasted image 20260328122550.png]]
1. 请求进入 `nova-api`
2. 通过消息队列分发任务 ~ **nova-api 把“开机请求”发到内部消息系统里**
3. `nova-scheduler` 选择宿主机 ~ **找出一台合适的 compute host 并把compute instance放到那台宿主机上**
4. `nova-conductor` 将一些状态、元数据、实例记录会写入数据库
5. `nova-compute` **用底层虚拟化层真正创建虚拟机**

---

## Storage

![[Pasted image 20260328133719.png]]
Different type of storage (different read and write speed) and therefore have different component to manage.

![[Pasted image 20260328133809.png]]

![[Pasted image 20260328133820.png]]

---

## Image

![[Pasted image 20260328135026.png]]

**Glance-api 是镜像服务的对外接口，负责镜像的查询、获取和存储请求。**

**Glance-registry 负责镜像元数据的存储、处理和检索**。

---

## Networking

![[Pasted image 20260328135556.png]]
- **Supports networking of OpenStack services**
    - 支持 OpenStack 各种服务的网络连接。
    - 简单说就是：**让实例和服务能联网、互联。**
- **Offers an API for users to define networks and the attachments into them**
    - 提供 API，让用户定义网络以及设备如何接入这些网络。
	- For example: **how a what is a firewall rule and how do configure your server to only allow certain incoming connections or certain outgoing connections, or allow any connections**
	- By default, every time you create a server, you only have one port open, which is port 22 for `ssh`

---

## Orchestration Service
![[Pasted image 20260328135922.png]]
Heat 是 OpenStack 的模板编排服务，用来通过 JSON/YAML 模板自动创建和管理一整套云资源，这组资源通常称为一个 stack。


