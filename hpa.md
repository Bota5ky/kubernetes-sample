### HPA

控制管理器通过每隔 horizontal-pod-autoscaler-sync-period 时间查询 metrics 的资源使用情况，常用于 Deployment，DaemonSet 无法使用

支持三种 metrics 类型：

- 预定义的 metrics，比如 CPU
- 自定义的 Pod metrics，以 raw value 的方式计算
- 自定义的 object metrics
  - 控制管理器开启 -horizontal-pod-autoscaler-use-rest-clients
  - 控制器的 -apiserver 指向 API Server Aggregator
  - 在 API Server Aggregator 中注册自定义的 metrics API


预定义资源限制：

```yaml
# Deployment
spec:
  template:
    spec:
      containers:
      - resources:
          limits:
            cpu: 200m
            memory: 128Mi
          requests:
            cpu: 100m # 基于这个配置计算百分比
            memory: 128Mi
```

创建 HPA：kubectl autoscale deploy deploy_name --cpu-percent=20 --min=2 --max=5

kubectl get hpa 获得 HPA 状态

kubectl top pod：需要安装 Metrics API

- 下载 metrics-server 组件配置文件 wget https://github.com/kubemetes-sigs/metricsserver/releases/latest/download/components.yaml -O metrics.server-components.yaml

- 修改镜像地址为国内的地址 sed -i's/k8s.gcr.io\/metrics-server/registry.cn-hangzhoualiyuncs.com\/google_containers/g' metrics-server-components.yaml

- 修改容器的 tls 配置，不验证 tls，在 containers 的 args 参数中增加`--kubelet-insecure-ts`参数
- 安装组件 kubectl apply -f metrics-server-components.yaml