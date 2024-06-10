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