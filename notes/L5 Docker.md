![[Pasted image 20260418220736.png]]
Docker instances can share a host OS, and not required hypervisor

![[Pasted image 20260418220928.png]]

![[Pasted image 20260418220940.png]]

![[Pasted image 20260418221007.png]]

- **Bare Metal（裸机）**: 应用程序直接运行在物理服务器的操作系统之上，没有任何虚拟化层。
    
- **Virtualized（虚拟化）**: 通过 Hypervisor 在物理机上创建多个独立的虚拟机（VM），每个虚拟机拥有完整的客体操作系统。
    
- **Containerized（容器化）**: 容器共享主机的操作系统内核，通过容器引擎实现应用间的轻量级隔离与部署。
    
- **Containerized on Virtualized（虚拟化上的容器化）**: 现代云端的主流做法，即在虚拟机内部部署容器，结合了虚拟机的安全隔离性与容器的灵活调度性。

![[Pasted image 20260421172510.png]]
**Docker 的核心优势在于** 将应用程序及其所有依赖项（库、配置文件等）打包在一个隔离的容器中，确保了在从开发到生产的任何操作系统上，**依赖关系（Dependency）保持高度一致性**。

![[Pasted image 20260421215639.png]]

![[Pasted image 20260422130843.png]]
docker的底层由containerd来实现，当运行docker run等cmd时，都是docker daemon call container d之后再运行containerd的底层指令。
k8s同理，都是使用containerd作为底层架构。

![[Pasted image 20260422131112.png]]
容器编排工具负责在大规模集群中自动化地部署、扩展、调度和确保容器的健康运行。
![[Pasted image 20260422131152.png]]
SWARM基本不再使用，但是compose在单个host很常用且广泛。
CMPOSE负责单个host多个containers， k8s负责多个host多个containers

## CMD
![[Pasted image 20260422133021.png]]

## Data
![[Pasted image 20260422141916.png]]
bind volume类似于openstack的attach volume，由user自定义path

---

## Docker and Networking

![[Pasted image 20260422142905.png]]
docker和local host共享一个ip地址
但docker desktop是存在于映射的虚拟网络中， 如果直接host会被refuse
![[Pasted image 20260422143647.png]]
![[Pasted image 20260422143755.png]]
Docker 通过驱动程序，在操作系统内核之上为容器构建虚拟的网络隔离边界或通信桥梁。
- **Bridge (桥接):** 默认模式，通过虚拟网桥实现容器间通信，并依靠端口映射与外界交流。
- **None (禁用):** 完全切断网络协议栈，为对安全性要求极高的离线任务提供极致隔离。
- **Overlay (覆盖):** 跨越物理主机边界，在集群之上构建一个让所有容器像在同一个局域网内的虚拟网络。
- **Macvlan (直连):** 赋予容器真实的 MAC 地址和物理局域网 IP，使其在网络中表现得像一台独立的物理机。

---

## Security
![[Pasted image 20260422145533.png]]
每条都值得仔细记

---

# CI/CD
![[Pasted image 20260422162754.png]]
## CI
![[Pasted image 20260422172920.png]]

## CD
![[Pasted image 20260422172939.png]]
## CI/CD pipeline
![[Pasted image 20260422173201.png]]
以 YAML 形式存在的 Pipeline 通常会定义以下几个核心要素：

- **触发条件 (Triggers/Events)：** 决定流水线什么时候跑。比如“当有人 push 代码到 `main` 分支时”。
    
- **阶段划分 (Stages/Jobs)：** 也就是你提到的 stage，通常按顺序定义为 `Build` (编译/构建) -> `Test` (跑单元测试/集成测试) -> `Deploy` (发布到服务器)。
    
- **具体步骤 (Steps/Commands)：** 告诉机器在每个 stage 里具体敲什么代码（比如运行 `npm install`，然后跑 `pytest` 等具体的 Bash 命令）。

EXAMPLE:
```yaml
# ==========================================
# 基础配置区
# ==========================================

# 1. 流水线名称：在 GitHub Actions 页面显示的标题
name: CI Pipeline

# 2. 触发机制：定义在什么情况下运行这条流水线
on:
  push:
    # 当有代码 push 到 main 分支时自动触发
    branches: [ "main" ]

# ==========================================
# 任务定义区
# ==========================================
jobs:
  # 定义一个名为 pipeline 的任务组
  pipeline:
    # 申请一台最新版 Ubuntu 的免费虚拟机作为运行环境
    runs-on: ubuntu-latest 
    
    steps:
      
      # 1. 基础依赖：拉取代码
      # 使用官方 actions/checkout 插件将代码克隆到虚拟机中
      - name: Checkout repository code
        uses: actions/checkout@v4

      # 2. 基础依赖：配置 Python 环境
      # 使用官方 actions/setup-python 插件安装指定版本的 Python
      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.10'
```

## The Pipeline Stages – Lint Check
![[Pasted image 20260422173601.png]]
EXAMPLE:
```yaml
      # ==========================================
      # 阶段一：Lint Check (代码规范检查)
      # 目标：检查代码格式，拦截不规范的代码进入后续测试
      # ==========================================
      
      # 1-1. 安装 Python 代码格式检查工具 Flake8
      - name: Install Flake8
        run: pip install flake8

      # 1-2. 执行检查
      # flake8 . ：检查当前目录下所有 Python 文件
      # --count ：统计发现的错误总数
      # --show-source ：在终端打印出具体报错的代码行
      - name: Run Lint Check
        run: flake8 . --count --show-source --statistics
```

## The Pipeline Stages – Dependency Check
![[Pasted image 20260422175218.png]]
```yaml
	  # ==========================================
      # 阶段二：Dependency Check (依赖安全检查 - 业界主流 Snyk)
      # ==========================================

      # 使用 Snyk 官方提供的 GitHub Action 插件进行依赖漏洞扫描。
      # 语法拆解：
      # snyk/actions 是 GitHub 上的开源仓库。
      # /python 是该仓库下针对 Python 环境的动作目录。
      # @master 表示使用该仓库主分支的最新代码 (注：生产环境建议锁死特定版本号如 @v4 以防突          发 bug)。
      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/python@master
        
        # env: 注入环境变量。在实际企业项目中，你需要去 Snyk 官网注册账号，
        # 获取一串 Token (电子通行密钥) 才能调用其扫描服务。
	    # secrets.SNYK_TOKEN: GitHub 的安全机制。你需要将 Token 存入仓库的 Secrets (密            码本) 中，
        # 运行时由流水线安全读取，避免明文写在代码中遭到泄露。
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        
        # with: 传递给该插件的具体配置参数。
        # args: --severity-threshold=high 表示过滤规则，这里指只拦截高危漏洞，忽略低危警            告。
        with:
          args: --severity-threshold=high
```

---

## The Pipeline Stages – Code Quality Analysis
![[Pasted image 20260422180235.png]]
```yaml
	  # ==========================================
      # 阶段三：Code Quality Analysis (代码质量分析)
      # 目标：进行深度静态代码扫描，识别 Bug、漏洞和代码坏味道
      # ==========================================

      # 使用 SonarQube 的云端版本 SonarCloud 进行扫描。
      # SonarCloud 会将扫描结果生成详细的 Dashboard 报告。
      - name: SonarCloud Code Analysis
        uses: SonarSource/sonarcloud-github-action@master
        env:
          # 与 Snyk 类似，需要访问控制权限。
          # GITHUB_TOKEN 是 GitHub 自动提供的内置令牌，无需手动设置。
          # SONAR_TOKEN 需要在 SonarCloud 网站注册获取并存入 Secrets。
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        with:
          # projectBaseDir 指定代码所在目录，默认为根目录
          args: >
            -Dsonar.projectKey=my_python_project_key
            -Dsonar.organization=my_github_org
            -Dsonar.python.version=3.10
```

SonarQube 就是一个强大的“代码核磁共振”。它不仅查格式，还能深入分析代码的逻辑结构，找出四类深层次问题：

1. **Bug**：明显的逻辑错误，比如变量未定义可能导致的崩溃。![[Pasted image 20260422180331.png]]
    
2. **Vulnerabilities**：代码逻辑层面写出的安全漏洞（比如 SQL 注入风险，注意这和前面的 Dependency Check 查第三方包漏洞不同，这是查你自己写的代码有没有漏洞）。![[Pasted image 20260422180348.png]]
    
3. **Security Hotspot**：有潜在安全风险的地方，比如在代码里明文写了密码，需要人工二次确认。![[Pasted image 20260422180408.png]]
    
4. **Code Smell (代码坏味道)**：虽然程序能跑，但写得很烂、很难维护的代码，比如定义了从未使用过的变量，或者一个函数写了 1000 行。![[Pasted image 20260422182509.png]]
---

## The Pipeline Stages – Unit Tests
![[Pasted image 20260422182551.png]]

```yaml
	  # ==========================================
      # 阶段四：Unit Tests (自定义单元测试与覆盖率检查)
      # 目标：执行开发者编写的单元测试，并拦截低覆盖率的代码
      # ==========================================

      # 4-1. 安装 Python 测试框架 pytest 及其覆盖率扩展包 pytest-cov
      - name: Install Pytest and Coverage plugin
        run: pip install pytest pytest-cov

      # 4-2. 自动发现并运行测试，同时进行覆盖率审计
      # pytest: 核心命令，自动扫描并执行你提交到仓库的所有 test_*.py 文件。
      # --cov=. : 统计当前项目录下所有代码的测试覆盖率。
      # --cov-fail-under=80 : 【企业级常见规则】如果整体代码的测试覆盖率低于 80%，即使所有         用例都 Pass，流水线也会强制报错拦截。
      - name: Run Custom Unit Tests and Check Coverage
        run: pytest --cov=. --cov-fail-under=80
```

### pytest 如何自动找到测试文件？

当流水线在虚拟机里敲下 `pytest` 这个单纯的命令时，你不需要告诉它去跑哪个具体的文件。它会自动以当前目录为起点，向下扫描所有的子文件夹，并严格寻找满足以下**默认命名规则**的文件和函数：
![[Pasted image 20260422183311.png]]
- **找文件**：它只寻找文件名以 `test_` 开头（例如 `test_calculator.py`）或者以 `_test.py` 结尾的文件。
    
- **找函数**：在找到这些文件后，它只会执行其中名字以 `test_` 开头的函数（例如 `def test_add():`）。
    
所有不符合这个命名规范的文件和函数，pytest 都会直接忽略，认为它们是普通的业务代码。

### 测试代码是如何与被测源码“匹配”的？

pytest **并不会**在底层自动把 `test_calculator.py` 和 `calculator.py` 强行绑定在一起。其实，哪怕你把测试文件取名叫 `test_math_stuff.py`，只要它以 `test_` 开头，pytest 依然会去跑它。

在 `test_calculator.py` 的开头，有一行极其关键的代码： `from calculator import add`。它们之间真正的“匹配”和关联，是靠 Python 代码里的 **Import (导入) 语句** 完成的


---

## The Pipeline Stages – Integration/E2E Tests
![[Pasted image 20260422182700.png]]

```yaml
	  # ==========================================
      # 阶段五：Integration & E2E Tests (集成测试与端到端测试)
      # 目标：测试组件间交互是否有缺陷 (集成)，以及模拟真实用户操作整个系统流程 (E2E)
      # ==========================================

      # 5-1. 执行集成测试 (Integration Tests)
      # 单元测试只管单个函数，集成测试则检查它们拼在一起能不能工作（比如后端接不接得上数据库）。
      # 实践中，通常会把这类耗时较长的测试放在单独的文件夹里，用 pytest 指定目录运行。
      - name: Run Integration Tests
        run: pytest tests/integration/

      # 5-2. 准备端到端测试环境 (以业界极度流行的 Playwright 为例)
      # E2E 测试的要求最高，它会直接启动一个真实的浏览器(无界面模式)去跑你的网页。
      # 这一步会安装 Playwright 及其依赖的浏览器内核（Chromium/Firefox/WebKit）。
      - name: Install Playwright Browsers
        run: |
          pip install playwright
          playwright install --with-deps

      # 5-3. 执行端到端测试 (End-to-End Tests)
      # 模拟真实用户的完整行为路径（比如：打开首页 -> 登录 -> 放入购物车 -> 结账）。
      # 只有当系统能够顺畅走完整个业务流，才算达到 PPT 里说的 "meets the specified                requirements"。
      - name: Run E2E Tests
        run: pytest tests/e2e/
```

 ---

## The Pipeline Stages – Pack the Software

![[Pasted image 20260422183941.png]]
```yaml
	  # ==========================================
      # 阶段六：Pack the Software (打包软件)
      # 目标：将经过测试的代码与其依赖环境彻底打包成一个标准化的 Docker 镜像，解决“在我电脑上         能跑，在服务器上报错”的环境差异问题。
      # ==========================================

      # 6-1. 设置 Docker 构建引擎
      # uses: docker/setup-buildx-action@v3
      # 引入 Docker 官方的 Buildx 插件。Buildx 是现代的构建工具，
      # 它比传统的 docker build 更快，并且支持高级缓存机制和多平台(如 ARM 和 AMD)架构构建。
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      # 6-2. 执行镜像构建
      # uses: docker/build-push-action@v5
      # 这是业界最标准、最常用的 Docker 官方 GitHub Action 动作插件。
      - name: Build Docker Image
        uses: docker/build-push-action@v5
        
        # with: 传递给构建动作的核心参数
        # context: . 表示将当前仓库的根目录作为构建上下文，插件会自动寻找目录下的                     Dockerfile 文件。
        # tags: 为构建出来的镜像打上名称和版本标签，latest 是默认的最新版本标识。
        # push: false 表示当前阶段仅在 CI 虚拟机本地完成打包验证，暂时不上传到 Docker Hub            等远端镜像仓库。 
        with:
          context: .
          tags: my-python-app:latest
          push: false
```

