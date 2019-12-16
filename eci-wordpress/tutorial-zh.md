# 使用 ASK + ECI 一分钟体验 WordPress

## 创建 Serverless Kubernetes 集群

您可以使用 Aliyun CLI 命令方便的创建集群，先切换到指定的 Region
```bash
switch-region cn-chengdu
```

```bash
aliyun cs POST /clusters --header "Content-Type=application/json" --body "$(cat create.json)"
```

其中 `create.json` 文件保存有创建 Serverless Kubernetes 集群的参数，您可以自定义来配置自己的集群。

- cluster_type：集群类型，Serverless Kubernetes 集群类型为 "Ask"
- name：集群名称
- nat_gateway：是否创建NAT网关
- private_zone：是否开启privateZone用于服务发现

创建成功后，您可以在控制台中看到执行完的输出，如下所示：

```
{
    "cluster_id": "c61cf530524474386a7a******",
    "request_id": "348D4C9C-9105-4A1B-A86E-******",
    "task_id": "T-5ad724ab94a2b109e*****"
}
```

其中 `cluster_id` 为您创建的集群的唯一 id。

您现在可以登录[容器服务控制台](https://cs.console.aliyun.com)查看通过 Aliyun CLI 创建的 Serverless Kubernetes 集群。

## 安装 WordPress

**注意：请确保上一步中创建的 Serverless Kubernetes 集群，已完成初始化（一般需要3~5分钟），再开始以下的操作**

使用 Cloud Shell 来管理上一步中创建中的 Serverless Kubernetes 集群
```bash
source use-k8s-cluster ${集群ID}
```

执行 WordPress 安装 yaml 文件
```bash
kubectl apply -f wordpress-all-in-one-pod.yaml
```

观察安装进度，直到 STATUS 为 Running
```bash
kubectl get pods
```

查询 EIP 地址
```bash
kubectl get -o json pod wordpress |grep "k8s.aliyun.com/allocated-eipAddress"
```

由于安全组默认没有开放 80 端口的访问，需要给安全组添加 80 端口的 ACL

首先获取 Serverless Kubernetes 集群自动创建的安全组，找到最近创建的以`alicloud-cs-auto-created`为命名前缀的安全组ID

```bash
aliyun ecs DescribeSecurityGroups|grep -B 1 'alicloud-cs-auto-created'|head -1
```

对安全组进行授权操作
```bash
aliyun ecs AuthorizeSecurityGroup --RegionId cn-chengdu --SecurityGroupId ${安全组ID} --IpProtocol tcp --PortRange 80/80 --SourceCidrIp 0.0.0.0/0 --Priority 100
```

## 使用 WordPress

在浏览器起输入上一步获取到的 EIP 地址，即可开始使用 WordPress