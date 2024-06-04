### Pod

`.spec.revisionHistoryLimit`：限制旧 RS 版本保留的数目，在回滚版本时可用

回退版本指定具体版本：kubectl rollout undo deployment/abc --to-revision=3

查看具体版本的信息：kubectl rollout history daemonset/abc --revision=3

### StatefulSet

- Headless Service：对于有状态服务的 DNS 管理

- volumeClaimTemplate：用于创建持久化卷的模板

`kubectl run -it --image busybox:1.28.4 dns-test --restart=Never --rm /bin/sh`进入工具容器查看 启动 pod 的网络状态：

- nslookup web-0.nginx，每个 Pod 的 DNS 格式为 statefulSetName-{0..N-1}.serviceName.namespace.svc.cluster.local
- Headless Service 和 StatefulSet 必须在相同的 namespace

会按名称序号扩缩容：

- kubectl scale statefulset web --replicas=5
- kubectl patch statefulset web -p '{"spec":{"replicas":3}}'

倒序镜像更新（目前不支持直接更新 image， 需要 patch 来间接实现）：

kubectl patch statefulset web --type='json' -p='[{"op": "replace", "path":"/spec/template/spec/containers/0/image", "value":"registry.k8s.io/nginx-slim:0.8"}]'

利用属性 updateStrategy.rollingUpdate.partition 实现灰度发布，只对>=partition的 pod 进行更新

updateStrategy.type: OnDelete 删除时才进行更新

级联删除：默认会同时删除 pods，非级联 kubectl delete sts web --cascade=orphan

### DaemonSet

fluentd 日志收集发送到 ES 中

根据 nodeSelector、nodeAffinity、podAffinity 筛选部署的 node，不设置默认部署到所有工作节点

```yaml
apiVersion: apps/v1 # 现在已不是extensions
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      app: logging # 对应的 DaemonSet pod 的标签
  template:
    metadata:
      labels:
        app: logging
        id: fluentd
      name: fluentd
    spec:
      nodeSelector:
        type: microservice
      containers:
      - name: fluentd-es
        image: agilestacks/fluentd-elasticsearch:v1.3.0
        env:
        - name: FLUENTD_ARGS
          value: -qq
        volumeMounts: # 加载数据卷，避免数据丢失
        - name: varlog # 数据卷的名字
          mountPath: /varlog # 将数据卷挂载到容器内的哪个目录
      volumes: # 定义数据卷
      - hostPath: # 数据卷类型，主机路径的模式，也就是与 node 共享目录
          path: /var/log # node 中的共享目录
        name: varlog # 定义的数据卷名称
```

### HPA

控制管理器通过每隔 horizontal-pod-aiyoscaler-sync-period 时间查询 metrics 的资源使用情况，常用于 Deployment，DaemonSet 无法使用

支持三种 metrics 类型：

- 预定义的 metrics，比如 CPU
- 自定义的 Pod metrics，以 raw value 的方式计算
- 自定义的 object metrics

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
            cpu: 100m
            memory: 128Mi
```

创建 HPA：kubectl autoscale deploy deploy_name --cpu-percent=20 --min=2 --max=5

kubectl get hpa 获得 HPA 状态
