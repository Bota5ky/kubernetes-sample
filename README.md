环境

```
VMware_17.5.2-23775571_Setup.exe
CentOS-7-x86_64-Minimal-2009.iso
kubernetes 1.23.6
docker-ce-20.10.12
```

系统设置

```bash
# 关闭防火墙，禁用开机启动
systemctl stop firewalld
systemctl disable firewalld
# 关闭selinux
sed -i 's/enforcing/disabled/' /etc/selinux/config #永久
setenforce 0 #临时，getenforce 0查看
# 禁用交换分区，关闭后需要重启虚拟机
swapoff -a #临时
sed -ri 's/.*swap.*/#&/' /etc/fstab #永久，删除或注释掉/etc/fstab里的swap设备的挂载命令
# 根据规划设置主机名
hostnamectl set-hostname <hostname>
# 在master添加hosts
cat >> /etc/hosts << EOF
192.168.239.128 k8s-master
192.168.113.121 k8s-node1
192.168.113.120 k8s-node2
EOF
# 将桥接的IPv4流量传递到iptables的链
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system #生效
# 时间同步，若同步失败就换国内时间服务器
yum install ntpdate -y
ntpdate time.windows.com
```

yum 安装 ifconfig 报错处理

```bash
vi /etc/sysconfig/network-scripts/ifcfg-ens33
# 修改
ONBOOT=yes
DNS1=114.114.114.114
# 重启网络
systemctl restart network
```

安装docker

```bash
# 移除旧版本的 Docker（如果有）
sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
# 安装必要的一些系统工具
yum install -y yum-utils device-mapper-persistent-data lvm2
# 设置 Docker 仓库
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 替换下载源为阿里源
sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
# 更新源
yum makecache fast
# 查看可安装版本
yum list docker-ce --showduplicates | sort -r
# 选择版本安装
sudo yum install -y docker-ce-20.10.12 docker-ce-cli-20.10.12 containerd.io
# 设置开机启动并启动Docker
systemctl enable docker && systemctl start docker
# 配置镜像下载加速，若kubelet启动失败，需要设置k8s、docker驱动器一致
cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors" : [
    "http://hub-mirror.c.163.com",
    "http://registry.docker-cn.com",
    "http://docker.mirrors.ustc.edu.cn"
  ],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF
# 重启生效
systemctl restart docker
docker info | grep 'Server Version'
```

配置yum源

```bash
cat  > /etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

安装kubeadm、kubelet、kubelet

```bash
yum install -y kubelet-1.23.6 kubeadm-1.23.6 kubectl-1.23.6 #卸载remove
systemctl enable kubelet
```

Master节点初始化

```bash
kubeadm init --apiserver-advertise-address=192.168.239.128 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.23.6 --service-cidr=10.96.0.0/12 --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=all
# 安装成功后
mkdir -p $HOME/.kube
# 拷贝kubectl使用的连接k8s认证文件到默认路径
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get nodes
# 重新初始化
kubeadm reset
```

加入kubernetes node，分别在 k8s-node1 和 k8s-node2 执行

```
kubeadm join 192.168.239.128:6443 --token cfmqdx.wg8u6c54o2pj3qlp --discovery-token-ca-cert-hash sha256:61fd6aca265465e44dbf0a10165a5d206777412b4d6d3eeed91de0cb756d226e
# 清空初始化后的记录或者token过期重新获取
kubeadm token create --print-join-command
```

安装CNI

```bash
curl https://calico-v3-25.netlify.app/archive/v3.25/manifests/calico.yaml -O -k
# 修改 calico.yaml 文件中的 CALICO_IPV4POOL_CIDR 配置，修改为与初始化的 cidr 相同。默认已注释，可以不改
# 删除镜像 docker.io/ 前缀，避免下载过慢导致失败
sed -i 's#docker.io/##g' calico.yaml
kubectl apply -f calico.yaml
```

在任意节点使用kubectl

```bash
# 将 master 节点中 /etc/kubernetes/admin.conf 拷贝到需要运行的服务器的 /etc/kubernetes 目录中
scp /etc/kubernetes/admin.conf root@k8s-node1:/etc/kubernetes
# 在对应服务器上配置环境变量
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bahs_profile
source ~/.bash_profile
```

kubectl 短命令

```bash
# 在~/.bashrc加入，source ~/.bashrc生效
alias k='kubectl'
complete -o default -F __start_kubectl k
source <(kubectl completion bash)
# 代码补全
yum -y install bash-completion
```

