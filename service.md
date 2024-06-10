### Service

Service 实现 k8s 集群内部网络调用、负载均衡（四层负载）

Ingress 实现将 k8s 内部服务暴露给外网访问的服务，ingress-nginx 反向代理、负载均衡（七层负载）

kubectl get endpoints

IP 绑定在 pause 容器上

<img src="C:/Users/Administrator/Documents/repos/kubernetes-sample/service.png" width="80%" />

```yaml
spec:
  ports:
  - port: 80 # service 自己的端口，在使用内网 ip 访问时使用
    targetPort: 80 # 目标 pod 的端口
    name: web # 为端口起个名字
    type: NodePort # 随机启动一个端口(30000-32767)，映射到 ports 中的端口，该端口是直接绑定在 node 上的，且集群中的每一个 node 都会绑定这个端口
# 也可以用于将服务暴露给外部访问，但是这种方式实际生产环境不推荐，效率较低，而且 Service 是四层负载，无法处理应用层如 http 协议的header
```

默认协议为 TCP

代理 k8s 外部服务：

- 编写 service 配置文件时，不指定 selector 属性
- 自己创建 endpoint

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  labels:
    app: external-svc # 与service一致
    name: external-svc # 与service一致
    namespace: default # 与service一致
subsets:
- addresses:
  - ip: 120.78.159.117 # 目标ip地址
  ports:
  - name: http # 端口名与service一致
    port: 80
    protocol: TCO
```

反向代理外部域名：

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: external-domain
    name: external-domain
spec:
  type: ExternalName
  externalName: www.wolfcode.cn
```

常用类型：

- ClusterIP 默认类型，可缺省

- ExternalName 返回定义的 CNAME 别名，可以配置为域名

- NodePort 会在所有安装了 kube-proxy 的节点都绑定一个端口，此端口可以代理至对应的 Pod，集群外部可以使用任意节点 ip +NodePort 的端口号访问到集群中对应 Pod 中的服务。

  当类型设置为 NodePort 后，可以在 ports 配置中增加 nodePort 配置指定端口，需要在下方的端口范围内，如果不指走会随机指定端口

  端口范围：30000~32767
  端口范围配置在 /usr/lib/systemd/system/kube-apiserver.service 文件中

- LoadBalancer 使用云服务商（阿里云、腾讯云等）提供的负载均衡器服务

