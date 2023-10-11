# 使用 ECI + ACK Serverless 一分钟体验 WordPress

进入WordPress目录
```bash
cd eci-wordpress
```

## 创建 Serverless Kubernetes 集群

您可以使用 Aliyun CLI 命令方便的创建集群

```bash
aliyun cs POST /clusters --header "Content-Type=application/json" --body "$(cat create.json)"
```

其中 `create.json` 文件保存有创建 Serverless Kubernetes 集群的参数，您可以自定义来配置自己的集群。

- cluster_type：集群类型，Serverless Kubernetes 集群类型为 "ManagedKubernetes"
- profile：集群标识，参数cluster_type取值为ManagedKubernetes，同时该参数配置为Serverless，表示创建ACK Serverless集群。
- name：集群名称
- region_id：集群所在地域ID
- endpoint_public_access：是否开启公网API Server
- snat_entry：是否在VPC中创建NAT网关并配置SNAT规则
- addons：Kubernetes集群安装的组件列表
- zoneid：集群所属地域的可用区ID，如果不指定vpcid和vswitch_ids的情况下，必须指定zoneid。

创建成功后，您可以在控制台中看到执行完的输出，如下所示：

```json
{
  "cluster_id": "c486508d6416045a9a434b0******",
  "instanceId": "c486508d6416045a9a434b0******",
  "request_id": "075417EF-8F86-51E6-******",
  "task_id": "T-6524fe49265d5c06******"
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
预期返回
```shell
"k8s.aliyun.com/allocated-eipAddress": "39.105.XX.XX"
```

由于安全组默认没有开放 80 端口的访问，需要给安全组添加 80 端口的 ACL

首先获取安全组ID

```bash
kubectl get -o json pod wordpress |grep "k8s.aliyun.com/eci-security-group"
```

预期返回
```shell
"k8s.aliyun.com/eci-security-group": "sg-2zef08a606ey91******"
```

对安全组进行授权操作
```bash
aliyun ecs AuthorizeSecurityGroup --RegionId ${Region ID} --SecurityGroupId ${安全组ID} --IpProtocol tcp --PortRange 80/80 --SourceCidrIp 0.0.0.0/0 --Priority 100
```
请根据实际替换上述命令的Region ID和安全组ID。命令示例如下：
```bash
aliyun ecs AuthorizeSecurityGroup --RegionId cn-hangzhou --SecurityGroupId sg-2zef08a606ey91******  --IpProtocol tcp --PortRange 80/80 --SourceCidrIp 0.0.0.0/0 --Priority 100
```

## 使用 WordPress

在浏览器起输入上一步获取到的 EIP 地址，即可开始使用 WordPress