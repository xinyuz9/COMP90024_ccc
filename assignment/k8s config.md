**Step 1: 创建 Fission Python 运行环境** 

```Bash
fission env create --name python-env --image fission/python-env --builder fission/python-builder
```

**Step 2: 在项目根目录创建并拆分那环境文件** 在本地建好 `config.env` 和 `secret.env` 并填入对应的值。

**Step 3: 将配置打入云端 ConfigMap**

```Bash
kubectl create configmap global-api-config --from-env-file=config.env
```

**Step 4: 将机密打入云端 Secret**

```Bash
kubectl create secret generic global-api-secrets --from-env-file=secret.env
```

---

**通过“本地文件即代码 (Infrastructure as Code)”来保证拓展性。**

未来如果要加入 Reddit 的抓取只需要：

1. 在本地的 `secret.env` 里加上一行 `REDDIT_CLIENT_SECRET=xxx`。
    
2. 在终端运行一条极其优雅的“覆盖/更新”命令，将本地文件重新打入 K8s：
    
```Bash
kubectl create secret generic global-api-secrets --from-env-file=secret.env --dry-run=client -o yaml | kubectl apply -f -
```
    
```bash
kubectl create configmap global-api-config --from-env-file=config.env -o yaml --dry-run=client | kubectl apply -f -
```

这个命令的意思是：“根据我最新的本地文件重新生成一份配置，并无缝替换掉云端的那一份。”这样你的 Fission 函数就能瞬间读取到新加的 Reddit 密码，极其优雅且高度可拓展！

---

![[Pasted image 20260507232812.png]]

---

```bash
fission function test --name harvest-bluesky
```

