## 1. 镜像与注册中心管理 (Images & Registry)

| **命令**            | **详细含义**                | **参数说明**                                                                                                                                                           | **示例**                             |
| ----------------- | ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------- |
| **docker login**  | 登录到指定的 Docker 镜像库 。     | `-u`: 指定用户名 ；`-p`: 指定密码或 PAT 。                                                                                                                                     | `docker login -u user123`          |
| **docker pull**   | 从注册中心下载镜像到本地 。          | `NAME[:TAG]`: 指定镜像名和标签（默认 latest） 。                                                                                                                                | `docker pull nginx:alpine`         |
| **docker images** | 列出本地存储的所有镜像 。           | 无必需参数，返回 ID、创建时间及大小 。                                                                                                                                              | `docker images`                    |
| **docker build**  | 根据 Dockerfile 自动化构建镜像 。 | `-t`: 为镜像打标签（格式为 name:tag） ；`.`: 指定构建上下文路径，这里是将当前文件夹下的**所有**内容（除非被排除）打包并发送给 Docker Daemon。Daemon 接收到了所有文件，但只有你在 Dockerfile 中显式使用 `COPY` 或 `ADD` 指令的文件，才会被真正写入镜像的层中 | `docker build -t myapp:v1 .`       |
| **docker tag**    | 为现有镜像创建新的标签（别名） 。       | `<SOURCE> <TARGET>`: 源镜像名与目标标签名 。                                                                                                                                  | `docker tag nginx myrepo/nginx:v1` |
| **docker push**   | 将本地镜像上传至注册中心 。          | `NAME[:TAG]`: 指定要推送的镜像标签 。                                                                                                                                         | `docker push myrepo/nginx:v1`      |

---

## 2. 容器生命周期控制 (Container Lifecycle)

| **命令**             | **详细含义**      | **参数说明**                                                         | **示例**                                      |
| ------------------ | ------------- | ---------------------------------------------------------------- | ------------------------------------------- |
| **docker run**     | 创建并启动一个新容器 。  | `-d`: (Detached) 后台运行容器 ；`-p`: 端口映射（宿主机:容器） ；`--name`: 自定义容器名称 。 | `docker run --name web -p 8080:80 -d nginx` |
| **docker ps**      | 查看容器状态列表 。    | 无参数默认显示运行中容器 ；`-a`: (All) 显示所有状态的容器 。                            | `docker ps -a`                              |
| **docker stop**    | 优雅地停止运行中的容器 。 | `CONTAINER`: 容器 ID 或名称 。                                         | `docker stop web`                           |
| **docker restart** | 重启容器 。        | `CONTAINER`: 容器 ID 或名称 。                                         | `docker restart web`                        |
| **docker rm**      | 删除已停止的容器 。    | `-f`: (Force) 强制删除运行中的容器 。                                       | `docker rm -f web`                          |
可以使用container id的前三位来代full id 来进行docker rm
![[Pasted image 20260422225655.png]]


---

## 3. 交互、存储与编排 (Operations & Orchestration)

| **命令**             | **详细含义**       | **参数说明**                                                          | **示例**                               |
| ------------------ | -------------- | ----------------------------------------------------------------- | ------------------------------------ |
| **docker exec**    | 在运行的容器中执行新命令 。 | `-ti`: `t` (TTY) 分配终端，`i` (Interactive) 保持交互 ；`-w`: 指定工作目录 。      | `docker exec -ti web bash`           |
| **docker volume**  | 管理持久化存储数据卷 。   | `create --name`: 创建命名卷 ；`docker run -v volume_name`: 在 run 时挂载卷 。 | `docker volume create --name htdocs` |
| **docker compose** | 定义并运行多容器应用 。   | `up -d`: 启动服务并在后台运行 ；`down`: 停止并移除所有资源 。                          | `docker compose up -d`               |
| **docker debug**   | 访问容器进行高级调试 。   | 即使在只读文件系统容器中也能工作 。                                                | `docker debug my-app`                |

当您创建一个**命名卷 (Named Volume)** 时，Docker 会在宿主机的指定根目录下自动创建一个子目录 。
- **默认根路径**: `/var/lib/docker/volumes/`
    
- **具体挂载点**: 对于名为 `htdocs` 的卷，其真实数据位于 `/var/lib/docker/volumes/htdocs/_data`。

---

## 4. 安全与维护 (Security & Analysis)

|**命令**|**详细含义**|**参数说明**|**示例**|
|---|---|---|---|
|**docker scout cves**|扫描镜像中的漏洞 (CVE) 。|`IMAGE`: 需要扫描的镜像名 。|`docker scout cves nginx:latest`|
|**docker scout quickview**|快速查看镜像安全摘要 。|提供漏洞数量和基础镜像状态建议 。|`docker scout quickview nginx`|
|**docker scout recommendations**|获取基础镜像升级建议 。|用于寻找修复漏洞的更安全版本 。|`docker scout recommendations nginx`|

---

![[Pasted image 20260422233841.png]]

---

# Dockerfile
![[Pasted image 20260423000323.png]]
Example:
```dockerfile
# 1. 基础镜像 (Base Image)：构建的起点 
FROM nginx:latest

# 2. 环境变量 (Environment Variable)：定义容器运行时的变量 
ENV WELCOME_STRING="nginx in Docker"

# 3. 工作目录 (Working Directory)：设置后续指令的执行路径 
WORKDIR /usr/share/nginx/html

# 4. 复制文件 (Copy)：将宿主机的脚本复制到镜像根目录 
COPY ["./entrypoint.sh", "/"]

# 5. 构建时运行 (Run at Build Time)：在打包镜像时执行的指令 
# 包括备份文件、设置脚本权限以及安装调试工具 
RUN cp index.html index_backup.html; \
    chmod +x /entrypoint.sh; \
    apt-get update && apt-get install -qy vim

# 6. 入口点 (Entrypoint)：容器启动时总是执行的脚本 
ENTRYPOINT ["/entrypoint.sh"]

# 7. 默认命令 (Command)：作为参数传递给 ENTRYPOINT 
# 这里的命令是启动 Nginx 并保持在前台运行 
CMD ["nginx", "-g", "daemon off;"]
```

**Example bash script**:   `entrypoint.sh`
```bash
#!/usr/bin/env sh

# 使用环境变量替换 index.html 中的字符串 
sed -i 's/Welcome to nginx!/Welcome to '"${WELCOME_STRING}"'!/g' /usr/share/nginx/html/index.html

# Make the ENTRYPOINT a pass through, then runs the command specified in CMD. By default, “$@” variable points to the command line arguments. 
exec "$@" 
# exec nginx -g “daemon off;” 
```

- **关于 COPY**：
    - `COPY` 指令在这里**只负责**把 `entrypoint.sh` 脚本从宿主机存入镜像根目录 。
    - 它**没有**备份 `index.html`。备份是由 `RUN cp` 命令在镜像内部文件系统操作完成的 。
        
- **关于 ENTRYPOINT**：
    - 它是容器的“第一指令” 。
    - 它调用的脚本会在 Nginx 服务正式开启前，完成所有的文本替换工作 。
    - 最后通过 `exec "$@"` 指令，顺滑地切换到真正的 Nginx 进程 。

通过 `ENTRYPOINT` 结合 `ENV`，你可以实现“一个镜像，无限种配置”。只要在 `docker run` 时传入不同的环境变量，容器就会显示不同的内容 。

在 Dockerfile 中写 `ENV WELCOME_STRING="nginx in Docker"` 的作用是：**如果你在启动时不额外说明，我们就用这个值** 。

但是，Docker 允许你在执行 `docker run` 时，通过参数直接**覆盖**这个环境变量，而完全不需要重新构建 (Build) 镜像。

|**维度**|**操作方式**|**是否需要重新 Build**|**结果 (index.html 内容)**|
|---|---|---|---|
|**默认状态**|`docker run my-nginx`|否|"Welcome to nginx in Docker!"|
|**运行时修改**|`docker run -e WELCOME_STRING="COMP90024" my-nginx`|**否**|"Welcome to COMP90024!"|

---

## Docker Compose
![[Pasted image 20260423132223.png]]
![[Pasted image 20260423132305.png]]

**Docker Compose** 允许你通过一个 YAML 配置文件定义并运行整个应用栈 。
例如example中

|**服务名称**|**镜像 (Image)**|**核心配置**|**功能描述**|
|---|---|---|---|
|**wordpress**|`wordpress:latest`|映射端口 `8080:80`|Web 应用主体，通过环境变量连接到 `db` 服务。|
|**db**|`mysql:8`|绑定挂载 `./mysql` 到 `/var/lib/mysql`|关系型数据库，用于存储文章、用户等持久化数据。|
|**phpmyadmin**|`phpmyadmin/phpmyadmin:latest`|平台 `linux/amd64`|基于 Web 的数据库管理工具，方便可视化操作 MySQL。|

---

## Docker Hardening

### Dockerfile 加固指令解析
![[Pasted image 20260423133235.png]]
为了实现“非 Root 用户运行”，Dockerfile 进行了大幅调整，以确保 `nginx` 用户拥有运行所需的权限。

| **指令/操作**                        | **详细含义与目的**                                                                                   |
| -------------------------------- | --------------------------------------------------------------------------------------------- |
| **COPY default.conf...**         | 将自定义的 Nginx 配置文件复制到配置目录。                                                                      |
| **touch /var/run/nginx.pid**     | 预先创建 PID 文件。在非 root 模式下，Nginx 需要在此写入进程 ID。                                                    |
| **chown -R nginx:nginx ...**     | **核心加固步骤**：将日志 (`/var/log/nginx`)、缓存、配置和网页目录的所有权递归授予 `nginx` 用户 。没有这一步，非 root 用户将无法写入日志或启动服务。 |
| **sed -i 's/user nginx;//g'...** | 修改 `/etc/nginx/nginx.conf`，删除其中的 `user` 指令。因为非 root 用户没有权限切换用户身份。                             |
| **mkdir -p ...**                 | 创建临时缓存目录（如 `client_temp`, `proxy_temp` 等），并赋予 `nginx` 用户权限。                                   |
| **USER nginx**                   | **身份切换**：将容器的执行环境从默认的 `root` 切换为低权限的 `nginx` 用户。                                              |

### 加固效果验证
![[Pasted image 20260423133523.png]]
通过上述修改，容器表现出极强的防御能力：
- **限制安装与修改**：
    - 由于不再是 root 用户，执行 `apt update` 或 `apt install` 会提示 `Permission denied` 
    - 无法删除系统关键文件（如 `/etc/issue`），系统会提示权限不足 。
        
- **目录访问控制**：
    - `nginx` 用户只能在被 `chown` 授权的目录下创建文件。在其他目录（如 `/usr/share/nginx`）尝试 `touch demo` 会被拦截 。
        
- **只读文件系统 (Read-only Filesystem)**：
    - 如果启动容器时开启了只读模式，即使攻击者获得了权限，尝试在任何地方写入文件都会收到 `Read-only file system` 报错