![[Pasted image 20260324113144.png]]

```bash
ssh xizhou9@spartan.hpc.unimelb.edu.au
```
# SPARTAN

## Storage

![[Pasted image 20260324113233.png]]

| **存储路径**        | **存储类型**                    | **特点**            | **适用场景**             |
| --------------- | --------------------------- | ----------------- | -------------------- |
| `/data/gpfs`    | **Capacity Tier** (10K SAS) | 容量大 (4.18PB)，成本适中 | 长期存放科研数据集            |
| `/data/scratch` | **Performance Tier** (NVMe) | 极速读写 (761TB)，低延迟  | 计算任务中的临时读写 (Scratch) |

**并行文件系统 (PFS)**： 通过 **数据条带化 (Data Striping)** - 将一个文件拆分成多个“块 (Chunks)”。将这些块分散存储在不同的物理磁盘和存储服务器上， 以允许数千台服务器（客户端）**同时读写**同一个大文件的不同部分，而不会产生严重的 IO 瓶颈。


## Architecture

![[Pasted image 20260324115403.png]]

|**组件**|**角色**|**关键功能**|
|---|---|---|
|**登录节点 (Logon)**|**大门**|用户接入、代码编译、作业提交。**禁止直接运行计算任务。**|
|**管理节点 (Mgmt)**|**大脑**|资源调度 (Slurm)、作业队列管理、集群健康监控。|
|**计算节点 (Worker)**|**苦力**|实际执行并行计算的硬件阵列，HPC 的算力核心。|
|**存储系统 (Storage)**|**仓库**|挂载于所有节点，支持大规模并行读写 (如 GPFS/NVMe)。|
|**高速网络 (Network)**|**神经**|低延迟互联（InfiniBand），负责节点间通讯与数据交换。|

### Workflow

1. **接入**：用户登录 **Logon Node**。
    
2. **调度**：提交作业至 **Management Node**（进入队列）。Once you've logged in, you've got an SSH session. You put your job into a queue, and that's normally  the job scheduling is done by management.
    
3. **分配**：调度器根据资源情况，将作业派发给空闲的 **Worker Nodes**。
    
4. **执行**：计算节点从 **Storage** 读取数据并开始并行计算。
    
5. **回传**：计算结果写回 **Storage**。


## Modules

![[Pasted image 20260324125651.png]]

**Module** 是一个动态的环境变量调节器，通过修改 `$PATH` 等路径让特定软件对当前会话可见。

它在脚本中按需挂载预编译的软件和库，确保计算任务拥有精确且优化的运行环境。

如果不显式加载 Module，系统默认为仅含Linux 系统自带的极其基础的工具（如最基础的 `ls`, `cd`, `cp` 和一个通常版本很旧的默认 Python/GCC“) 的裸机”状。

它与容器（Docker）不同，是一种更轻量、针对集群硬件深度优化的“即插即用”工具管理方式。
	
- **Docker**：在 Linux 内核上封装了一层轻量级的“文件系统墙”，它带走了整个操作系统的根目录（库、配置、二进制文件），像一个**自给自足的集装箱**。
    
- **Module**：直接运行在宿主系统中，它不造墙，只是通过环境变量（如 `$PATH`）告诉你：“你要的 3.10 版本 Python 在 `/opt/modulefiles/python/3.10` 这个文件夹里。


### CMD
 
![[Pasted image 20260324133253.png]]

![[Pasted image 20260324133314.png]]



## Job submission/scheduling

![[Pasted image 20260324142732.png]]
## SLURM script

![[Pasted image 20260324143058.png]]
**Slurm** (Simple Linux Utility for Resource Management) 是整个集群的**软件系统**。
- **角色**：它是资源调度器（Scheduler）。
    
- **职责**：
    - 监控数千个计算节点的死活。
    - 管理所有人的作业排队（Queueing）。
    - 强制执行规则（比如你最多只能跑 2 天，不能超过 1TB 内存）。


**sbatch** 是 Slurm 提供的一个**命令行指令**，专门用于提交**非交互式**（后台运行）的作业脚本。

- **角色**：它是用户与 Slurm 沟通的“翻译官”。
- **流程**：
    1. 你写好一个包含 `#SBATCH` 指令的脚本（如 `test.sh`）。
    2. 你执行 `sbatch test.sh`。
    3. **sbatch** 把你的需求（要多少 CPU、跑多久）发给 **Slurm**。
    4. **Slurm** 收到后，把你的作业放进队列里排队。


![[Pasted image 20260324150100.png]]

```bash
#!/bin/bash
#SBATCH --job-name=testPy           # 作业名称
#SBATCH --time=00:10:00             # 运行时间限制 (时:分:秒)
#SBATCH --nodes=1                   # 请求的物理节点数量
#SBATCH --ntasks=4                  # 总任务数 (通常对应 MPI 进程数)
#SBATCH --cpus-per-task=1           # 每个任务分配的 CPU 核心数
#SBATCH --output=res_%j.out         # 标准输出文件 (%j 会替换为作业 ID)

# 1. 环境清理：确保没有冲突的旧模块
module purge 

# 2. 动态加载依赖：按需挂载预编译好的软件栈
module load GCC/11.3.0
module load OpenMPI/4.1.4
module load mpi4py/3.1.4

# 3. 执行计算：使用 mpirun 启动并行 Python 任务
mpirun python testPy.py
```

- **SBATCH 指令**：这些是以 `#` 开头的特殊注释，由 **Management Node** 的调度器（Slurm）读取，用于决定在哪里以及如何分配资源。
    
- **Module 层**：
    - `module purge`：擦除当前登录环境的所有变量，建立“干净”的基准。
    - `module load`：将集群中预装的编译器（GCC）、并行库（OpenMPI）和工具包（mpi4py）**动态映射**到该作业的运行环境中。
        
- **执行层**：`mpirun` 会根据 `#SBATCH --ntasks=4` 的设置，同时启动 4 个 Python 实例进行协同计算。

![[Pasted image 20260324151341.png]]
 
 1. `testPy2.sh` Serial Execution: 即使你在 `#SBATCH` 中申请了 4 个任务（ntasks=4），如果你只写 `python testPy.py`：
	- **行为**：系统只会启动 **1 个** Python 进程。
	- **后果**：剩下的 3 个任务位（Slots）会处于闲置状态，造成资源浪费。
	- **结论**：普通的命令无法自动识别 Slurm 分配的多任务环境。

2. `testPy3.sh`：Parallel Launchers

| **启动命令**      | **描述**                                                    |
| ------------- | --------------------------------------------------------- |
| **`mpirun`**  | 最通用的 MPI 启动器，由 OpenMPI 等库提供。                              |
| **`mpiexec`** | 与 `mpirun` 类似，是 MPI 标准定义的启动命令。                            |
| **`srun`**    | **Slurm 原生启动器**（推荐）。它能直接读取 `#SBATCH` 的参数，自动在分配的所有节点上启动进程。 |


## Job management

![[Pasted image 20260324153205.png]]

## Other Scenarios

![[Pasted image 20260324153751.png]]

### Array Jobs

**场景**：当你需要用同一个程序处理 100 个不同的数据集（如 `data_1.csv` 到 `data_100.csv`）时，不需要写 100 个脚本。

#### 核心机制

- **`#SBATCH --array=1-10`**：告诉 Slurm 同时提交 10 个子任务。
    
- **内置变量**：`${SLURM_ARRAY_TASK_ID}`。每个子任务会自动获得一个唯一的 ID。
    
- **示例代码**：
    
    Bash
    
    ```
    #SBATCH --array=1-10
    # 系统会自动运行 10 次，每次 ID 不同
    python myapp.py dataset_${SLURM_ARRAY_TASK_ID}.csv
    ```
    

### Job Dependencies

**场景**：当任务之间有先后顺序时（如：必须先完成“数据预处理”，才能开始“模型训练”）。

#### 常用指令 (`--dependency`)

| **指令**                 | **含义**                         |
| ---------------------- | ------------------------------ |
| **`afterok:jobid`**    | **最常用**。仅在指定作业成功完成（返回码 0）后才开始。 |
| **`afterany:jobid`**   | 只要指定作业结束（无论成功或失败）就运行。          |
| **`afternotok:jobid`** | 仅在指定作业失败后才开始。                  |

### 示例场景

1. **提交任务 A**：得到 Job ID `12345`。
2. **提交任务 B**：要求在 A 之后。
    
```Bash
   #SBATCH --dependency=afterok:12345
```


### Interactive Jobs

![[Pasted image 20260324154454.png]]

如果你需要实时调试代码、运行交互式命令，但又不想“杀死”登录节点，你需要将自己的终端**直接连接到计算节点**。

#### 核心命令：`sinteractive`

这是 SPARTAN 等系统提供的快捷命令，本质上是申请资源后立即为你打开一个计算节点的 Shell。

| **命令示例**                 | **含义**                      |
| ------------------------ | --------------------------- |
| `sinteractive --nodes=1` | 申请 **1 个** 计算节点进行实时交互。      |
| `sinteractive --nodes=2` | 申请 **2 个** 节点（适合测试跨节点并行程序）。 |

---

## Shared Memory Jobs

![[Pasted image 20260324155023.png]]


| **技术**      | **全称**                             | **并行类型** | **核心机制**               | **硬件限制**              |
| ----------- | ---------------------------------- | -------- | ---------------------- | --------------------- |
| **OpenMPI** | Open **Message Passing** Interface | **多进程**  | 消息传递 (Message Passing) | **跨节点**（可跑在成千上万台服务器上） |
| **OpenMP**  | Open **Multi-Processing**          | **多线程**  | 共享内存 (Shared Memory)   | **单节点**（受限于单台服务器的核心数） |

### Example

![[Pasted image 20260324155130.png]]

#### 代码实现

- **`#include "omp.h"`**：引入 OpenMP 函数库（如获取线程 ID）。
    
- **`#pragma omp parallel sections`**：定义一个并行区域，内部的每个 `section` 将由不同的线程并发执行。
    
- **`omp_get_thread_num()`**：返回当前执行线程的编号（ID）。
    

#### 编译指令
必须在编译时添加 `-fopenmp` 标志，否则编译器会忽略所有并行指令，按串行执行。

```Bash
gcc -fopenmp hello3omp.c -o hello3omp
```

#### Slurm 脚本配置
对于 OpenMP 作业，重点是**单节点、多 CPU 核心**的分配。

```Bash
#SBATCH --nodes=1              # 必须为 1，因为 OpenMP 无法跨节点
#SBATCH --ntasks=1             # 仅需 1 个任务实例（进程）
#SBATCH --cpus-per-task=2      # 每个任务分配的 CPU 核心数
```

#### 环境变量控制

在脚本中，使用 `export OMP_NUM_THREADS` 来明确指定程序运行时的线程数：

```Bash
export OMP_NUM_THREADS=8       # 告诉程序启动 8 个线程
./hello3omp
```

> **注意**：通常 `OMP_NUM_THREADS` 应等于或小于 `--cpus-per-task` 申请的数量，以获得最佳性能。 


