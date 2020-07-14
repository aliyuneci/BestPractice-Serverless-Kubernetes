### 1. 准备ASK集群
https://cs.console.aliyun.com/?spm=5176.eciconsole.0.0.68254a9cNv12zh#/k8s/cluster/createV2/serverless
容器服务控制台创建标准serverless k8s集群


### 2. 准备PV/PVC
准备两个nas盘，一个做gitlab runner cache，一个做maven仓库，请自行替换nas server地址和path 

``` shell
kubectl apply -f mvn-pv.yaml
kubectl apply -f mvn-pvc.yaml
kubectl apply -f nas-pv.yaml
kubectl apply -f nas-pvc.yaml
```

### 3. 准备Secret
* kubeconfig里的证书公私钥拷贝到secret中，secret.yaml
``` shell
kubectl apply -f secret.yaml
```

* docker-registry的认证信息，ECI支持免密拉取，但是push docker image 还是要用到
``` shell
kubectl create secret docker-registry registry-auth-secret --docker-server=registry.cn-hangzhou.aliyuncs.com --docker-username=${xxx} --docker-password=${xxx}
```

查看生成的secret可以用以下命令
``` shell
kubectl get secret registry-auth-secret --output=yaml
```

### 4. 准备ConfigMap
把gitlab runner 的url、token，ask集群的api server地址拷贝到config.yaml
``` shell
kubectl apply -f config-map.yaml
```

### 5. 准备imageCache(可选，节省镜像拉取时间)
目前ASK默认安装了 imagecache-crd，可以用以下命令查询，如果没有可以自己安装
``` shell
# 查看image cache crd 是否安转
kubectl get crd
# 安装image cache crd
kubectl apply -f imagecache-crd.yaml
# 制作imagecache
kubectl apply -f imagecache.yaml
```

### 6. 部署gitlab runner
``` shell
kubectl apply -f gitlab-runner-deployment.yaml
```

### 7. 导入git repo，java demo见 java-demo 目录
