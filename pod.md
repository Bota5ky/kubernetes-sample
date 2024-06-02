### Pod

- StartupProbe：适用于启动时间较长的应用程序，确保启动过程不会被错误地重启

- ReadinessProbe：未就绪不会接收外部流量

- LivenessProbe：用于检测容器是否仍然活着（即容器是否在健康状态下运行），如果 liveness probe 失败，Kubernetes 将会重新启动该容器。这个机制帮助确保应用程序始终处于可用状态

如果同时配置了 StartupProbe 和 ReadinessProbe，在 StartupProbe 成功之前，ReadinessProbe 和 LivenessProbe 都不会生效。一旦 StartupProbe 成功，后续的探测将由 ReadinessProbe 和 LivenessProbe 接管。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-http
spec:
  containers:
  - name: myapp
    image: myapp:latest
    livenessProbe:
      httpGet: # 还有tcpSocket（连接端口）、exec.command（执行返回0表示健康）形式
        path: /healthz
        port: 8080 # 返回200-400之间的状态码，则认为健康
    initialDelaySeconds: 3 # 初始化时间
    timeoutSeconds: 2 # 超时时间
    periodSeconds: 3 # 监测间隔时间
    successThreshold: 1 # 检查1次成功就表示成功
    failureThreshold: 2 # 检查2次失败就表示失败
```

Pod 的生命周期

![Pod生命周期](https://isekiro.com/kubernetes%E5%9F%BA%E7%A1%80-pod%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%92%8C%E7%8A%B6%E6%80%81/pod%E7%8A%B6%E6%80%81%E5%BC%82%E5%B8%B8%E5%9C%BA%E6%99%AF.png)

Pod 退出流程

- Endpoint 删除 pod 的 ip 地址

- Pod 变成 Terminating 状态：terminationGracePeriodSeconds: 30 中止前宽限期

- 执行 preStop 的指令

### Deployment

set 命令：kubectl set image deployment/nginx busybox=busybox nginx=nginx:1.9.1

`.spec.revisionHistoryLimit`：限制旧 RS 版本保留的数目，在回滚版本时可用

回退版本指定具体版本：kubectl rollout undo deployment/abc --to-revision=3

查看具体版本的信息：kubectl rollout history daemonset/abc --revision=3

### StatefulSet

- Headless Service：对于有状态服务的 DNS 管理

- volumeClaimTemplate：用于创建持久化卷的模板

