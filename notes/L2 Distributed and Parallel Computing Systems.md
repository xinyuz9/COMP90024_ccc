# Computer Scaling

## Vertical
![[Pasted image 20260315143054.png]]
Stop now, due to physical limits
## Horizontal
![[Pasted image 20260315143116.png]]
与其继续追求单个 chip 更快，现在通过增加更多普通机器并让它们并行运行，来提升整体 compute capacity

# Amdahl's Law
![[Pasted image 20260315145356.png]]
Need to aware that Amdahl's law greatly simplifies how much faster in principle it could go, because in reality you're adding more code into your system to deal with that, which you never have before.
![[Pasted image 20260315150205.png]]

# Gustafson-Barsis's Law

![[Pasted image 20260315154331.png]]

Gustafsson Barsis's laws focuses on: How much more can I do if I had all  these compute resources?

![[Pasted image 20260315154843.png]]

Whereas Gustavsson Bossy's laws is it can be formulated as If I ever have an average speed of 90 miles an hour, and I start travelling around to Broome or Sydney or wherever, as long as I can go longer and faster, I will eventually achieve a certain speed.


---

# Computer Architecture

![[Pasted image 20260315164317.png]]

Hardware 层包括 `CPU`、`Memory`、`I/O` systems 和 Permanent storage， Operating System 位于硬件之上负责管理这些资源并为程序提供运行环境，应用程序再运行在 OS 之上；整体性能取决于这些组件之间的协同与平衡。


## Different Type of Computer Arch

![[Pasted image 20260315182448.png]]

### SISD
![[Pasted image 20260315182419.png]]
You take a piece of data from a data pool. You take an instruction, you apply it to that, you
get the next one, you get the next one, and it was very much a single data process.

## MISD
![[Pasted image 20260315182557.png]]
Fault tolerant systems like the space shuttle, where you have a piece of data, and it has to be
processed and checked and checked and checked by multiple instructions applied to it for making sure that there was not an error, and if there was not an error, did it do this, did it do that etc. etc.?

You're processing all of that using the same piece of data coming through to guarantee fault tolerance and we don't have that very much anymore, but for certain safety critical systems, it's still there.


## SIMD
![[Pasted image 20260315183309.png]]
The data's coming in and it's being processed using the same um instruction across the different data streams coming through. Again, it's not as common now.

## MIMD
![[Pasted image 20260315193328.png]]
What's happening now.

---

# Approaches for Parallelism

![[Pasted image 20260315193440.png]]
There's lots of different ways to do this stuff. You can do it in hardware, you can do it in software, you can do it at the operating system level, lots of different ways to implement parallel systems and using multiple cores.


## Explicit vs Implicit Parallelization

![[Pasted image 20260315193830.png]]
**Explicit parallelism**  programmer 自己使用 MPI / OpenMP / threading 等工具写并行程序， 需要自己负责：  task decomposition  mapping tasks to processors  communication  synchronization  更灵活，但更复杂，容易出现 race condition 和不稳定 bug  

**Important**  
- 写了并行代码 ≠ 已经真正并行运行  
- You've implemented a system that in principle has all the features to run in parallel, but **you have to now tell the job scheduler on Spartan to make it run in parallel for you.**

---

## Open Multi-Processing (OpenMP)

![[Pasted image 20260315204942.png]]
- **Parallel construct**  
	- `#pragma omp parallel`  
	- 定义一个 parallel region，使多个 threads 同时执行这段代码  
  
- **Work-sharing constructs**  
	- 用于把工作分配给不同 threads  
	
	- `#pragma omp for`  
		- 将 loop iterations 分配给多个 threads  
	  
	- `#pragma omp sections`  
		- 将不同代码段分配给不同 threads  
	  
	- `#pragma omp single`  
		- 指定某段代码只由一个 thread 执行  
	  
	- `#pragma omp task`  
		- 动态创建 tasks，由 threads 执行

![[Pasted image 20260315205019.png]]
More details in workshop

---

## Message Passing Interface (MPI)

![[Pasted image 20260315205244.png]]
- `MPI_Init`  
	- 初始化 MPI 环境，开始并行计算  
- `MPI_Finalize`  
	- 结束 MPI 环境，终止计算  
- `MPI_Comm_size`  
	- 获取当前 communicator 中的 **process 总数**  
- `MPI_Comm_rank`  
	- 获取当前 process 的 **rank（进程编号）**  
- `MPI_Send`  
	- 发送 message  
- `MPI_Recv`  
	- 接收 message

Main Process
```
同一个 MPI program 可以在不同数量的 processes 上运行  
程序运行后，会先通过：  
	`MPI_Comm_size` 知道当前有多少 processes  
	`MPI_Comm_rank` 知道自己是哪一个 process  
然后代码根据 `size` 和 `rank` 决定各自要做什么  
常见模式：  
	rank 0 作为 master / coordinator  
	其他 ranks 负责具体工作  
	workers 完成后把结果 send 回 rank 0  
	rank 0 再做 aggregation
```

### Example
![[Pasted image 20260315211517.png]]
1. `MPI_Init(&argc, &argv);`  
	- 初始化 MPI 环境  
  
2. `MPI_Comm_rank(MPI_COMM_WORLD, &rank);`  
	- 获取当前 process 的 rank  
  
3. `MPI_Comm_size(MPI_COMM_WORLD, &size);`  
	- `MPI_COMM_WORLD` 是 **MPI 库里预先定义好的 communicator 常量**。 可以当作一个默认的通讯组包含了这次 MPI 程序启动时的 **所有 processes**
	- 获取总 process 数量  
    
4. `MPI_Finalize();`  
	- 结束 MPI 环境


---

# Hardware Parallelization

![[Pasted image 20260315224740.png]]

We don't want is to have conflicts where two processes are running trying to access the same information in cache at the same time, one of them gets access and the other one reads the data, but it's actually out of date now, this variable's been updated, this has read the wrong one, that's a conflict.

![[Pasted image 20260315224832.png]]
 
 ---

# Operating System Parallelism Approaches

![[Pasted image 20260315230302.png]]
**Interleaved**: It's not these things are happening at the same time, for me it looks like they are, but actually just time slicing. 

不是所有步骤都真的同时发生，而是cpu scheduling进行快速任务交替

---

# Software Parallelism Approaches

![[Pasted image 20260315233737.png]]
**Live lock** is a situation where both processes trying to take action to prevent deadlock .
But the actions you take are causing deadlock, so you're trying to correct the behavior, but by moving and changing direction, both at the same time, you're both stalled.

---

# Data Parallelism

![[Pasted image 20260315234308.png]]



数据并行的核心是在**分布式数据与分布式文件系统**上处理大规模数据，但这会带来 **Consistency、Availability、Partition tolerance** 之间的权衡问题。

---

# (Some) Erroneous Assumptions of Distributed Systems

![[Pasted image 20260316103519.png]]
## The network is reliable
![[Pasted image 20260316103823.png]]
- 不能假设数据一定：  
	- 成功到达  
	- 按发送顺序到达  
	- 保持不损坏  

## Latency is zero  
![[Pasted image 20260316103909.png]]
![[Pasted image 20260316103943.png]]
- 数据不可能“立刻”到达  
- 跨城市、跨国家、跨多个 hops 时，latency 会很明显  


  
##### 带宽是无限的（Bandwidth is infinite）
![[Pasted image 20260316104021.png]]
- 可传输的数据量始终受限  
- 在 big data 场景下，数据移动本身就是瓶颈  

## The network is secure
![[Pasted image 20260316105014.png]]
- 不能默认网络环境可信  
- 需要防范：  
	- password attacks  
	- SQL injection  
	- DDoS  
	- man-in-the-middle  
	- spoofing  
	- malware / brute force  
- 因此安全必须是系统设计的一部分  
  
## Topology 不会变化（Topology doesn't change）  
![[Pasted image 20260316104928.png]]
- 节点、IP、route、latency、service availability 都可能变化  
- “某个节点一直稳定存在”不是可靠前提  
- distributed systems 必须面对：  
	- node join / leave  
	- route changes  
	- service failures  
	  
## There is one administrator
- 现实中很多 distributed systems 跨多个组织 / 部门 / 管理域  
- 不同部分可能有不同 policy、权限和配置  
- 因而 coordination 和 governance 更复杂  
  
## 传输成本是 0（Transport cost is zero）  
- 数据传输并不免费  
- 成本可能来自：  
- 云服务流量费用  
- 跨网络传输费用  
- 上传/下载限额  
- 所以数据移动不仅是技术问题，也是经济问题  
  
## The network is homogeneous
- 节点与链路并不统一  
- 可能存在差异：  
- CPU 性能  
- memory / storage  
- network speed  
- reliability  
- operating environment  
- 因而系统不能假设所有机器表现一致  
  
## Time is ubiquitous  
- 不能假设所有机器的 clock 完全一致  
- 实际会有：  
	- clock drift  
	- skew  
	- synchronization error  
- 即使使用 NTP，也只是近似同步  
- 对 ordering、coordination、金融系统等非常重要