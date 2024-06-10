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

