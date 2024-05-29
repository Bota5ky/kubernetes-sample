Pod 的生命周期

![Pod生命周期](https://isekiro.com/kubernetes%E5%9F%BA%E7%A1%80-pod%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E5%92%8C%E7%8A%B6%E6%80%81/pod%E7%8A%B6%E6%80%81%E5%BC%82%E5%B8%B8%E5%9C%BA%E6%99%AF.png)

Pod 退出流程

- Endpoint 删除 pod 的 ip 地址
- Pod 变成 Terminating 状态：terminationGracePeriodSeconds: 30 中止前宽限期
- 执行 preStop 的指令
