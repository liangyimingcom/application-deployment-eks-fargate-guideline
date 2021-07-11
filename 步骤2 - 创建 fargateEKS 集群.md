

# 创建 EKS 集群



在创建 EKS 集群之前，我们还需要安装 eksctl 和 kubectl 工具。



## 安装 kubectl

```bash
sudo curl --silent --location -o /usr/local/bin/kubectl \
  https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl

sudo chmod +x /usr/local/bin/kubectl
```

参考文档：https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

验证 kubectl 安装结果：

```bash
kubectl version --client
```



## 安装 eksctl

[eksctl](https://eksctl.io/) 是 Amazon EKS 的官方管理工具, Go 语言实现, 底层通过 CloudFormation 对 AWS 资源进行管理。

安装命令:

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv -v /tmp/eksctl /usr/local/bin
```

参考文档：https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html

验证 eksctl 安装结果:

```bash
eksctl version
```



## 通过 eksctl 创建 EKS 集群（指定的VPC下创建集群）

下面的命令会创建一个名字为 **cluster-wms-prod-v1** 的 EKS 集群, 这个 EKS 集群会创建一个默认的 [Fargate Profile](https://docs.aws.amazon.com/eks/latest/userguide/fargate-profile.html)，同时也会创建 EC2 工作节点。

**由于是在指定的VPC下创建集群，避免了VPC子网没有正确打标签（tag）的问题。**后续创建的 Fargate 任务会运行在这个新创建的 VPC 中。也可以通过配置指定使用已有的 VPC。



在Cloud9上传“fargate-existing-vpc-wms-prod-v1.yaml”后运行，创建生产环境的 fargate EKS：

```bash
eksctl create cluster -f fargate-existing-vpc-wms-prod-v1.yaml
```



整个 EKS 集群创建过程大约 15 分钟, 输出信息如下图所示:

```bash
2021-07-11 16:54:10 [ℹ]  eksctl version 0.55.0
2021-07-11 16:54:10 [ℹ]  using region me-south-1
2021-07-11 16:54:10 [✔]  using existing VPC (vpc-0f62b217700e212ab) and subnets (private:map[me-south-1a:{subnet-09d49c0dc4f8fa13c me-south-1a 10.163.76.128/26} me-south-1b:{subnet-0da4b3753f5ff5ecb me-south-1b 10.163.76.192/26}] public:map[me-south-1a:{subnet-07665affc9af9f354 me-south-1a 10.163.76.0/26} me-south-1b:{subnet-0698927b9996123b5 me-south-1b 10.163.76.64/26}])
2021-07-11 16:54:10 [!]  custom VPC/subnets will be used; if resulting cluster doesn't function as expected, make sure to review the configuration of VPC/subnets
2021-07-11 16:54:10 [ℹ]  using Kubernetes version 1.19
2021-07-11 16:54:10 [ℹ]  creating EKS cluster "cluster-wms-prod-v1" in "me-south-1" region with Fargate profile
2021-07-11 16:54:10 [ℹ]  will create a CloudFormation stack for cluster itself and 0 nodegroup stack(s)
2021-07-11 16:54:10 [ℹ]  will create a CloudFormation stack for cluster itself and 0 managed nodegroup stack(s)
2021-07-11 16:54:10 [ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=me-south-1 --cluster=cluster-wms-prod-v1'
2021-07-11 16:54:10 [ℹ]  CloudWatch logging will not be enabled for cluster "cluster-wms-prod-v1" in "me-south-1"
2021-07-11 16:54:10 [ℹ]  you can enable it with 'eksctl utils update-cluster-logging --enable-types={SPECIFY-YOUR-LOG-TYPES-HERE (e.g. all)} --region=me-south-1 --cluster=cluster-wms-prod-v1'
2021-07-11 16:54:10 [ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "cluster-wms-prod-v1" in "me-south-1"
2021-07-11 16:54:10 [ℹ]  2 sequential tasks: { create cluster control plane "cluster-wms-prod-v1", 2 sequential sub-tasks: { 5 sequential sub-tasks: { wait for control plane to become ready, create fargate profiles, associate IAM OIDC provider, 2 sequential sub-tasks: { create IAM role for serviceaccount "kube-system/aws-node", create serviceaccount "kube-system/aws-node" }, restart daemonset "kube-system/aws-node" }, 1 task: { create addons } } }
2021-07-11 16:54:10 [ℹ]  building cluster stack "eksctl-cluster-wms-prod-v1-cluster"
2021-07-11 16:54:10 [ℹ]  deploying stack "eksctl-cluster-wms-prod-v1-cluster"
2021-07-11 16:54:40 [ℹ]  waiting for CloudFormation stack "eksctl-cluster-wms-prod-v1-cluster"
2021-07-11 16:55:10 [ℹ]  waiting for CloudFormation stack "eksctl-cluster-wms-prod-v1-cluster"
2021-07-11 16:56:10 [ℹ]  waiting for CloudFormation stack "eksctl-cluster-wms-prod-v1-cluster"
2021-07-11 16:57:10 [ℹ]  waiting for CloudFormation stack "eksctl-cluster-wms-prod-v1-cluster"
2021-07-11 16:58:10 [ℹ]  waiting for CloudFormation stack "eksctl-cluster-wms-prod-v1-cluster"
2021-07-11 16:59:10 [ℹ]  waiting for CloudFormation stack "eksctl-cluster-wms-prod-v1-cluster"
2021-07-11 17:00:10 [ℹ]  waiting for CloudFormation stack "eksctl-cluster-wms-prod-v1-cluster"
2021-07-11 17:01:10 [ℹ]  waiting for CloudFormation stack "eksctl-cluster-wms-prod-v1-cluster"
2021-07-11 17:02:11 [ℹ]  waiting for CloudFormation stack "eksctl-cluster-wms-prod-v1-cluster"
2021-07-11 17:03:11 [ℹ]  waiting for CloudFormation stack "eksctl-cluster-wms-prod-v1-cluster"
2021-07-11 17:04:11 [ℹ]  waiting for CloudFormation stack "eksctl-cluster-wms-prod-v1-cluster"
2021-07-11 17:06:11 [ℹ]  creating Fargate profile "fp-default" on EKS cluster "cluster-wms-prod-v1"
2021-07-11 17:08:20 [ℹ]  created Fargate profile "fp-default" on EKS cluster "cluster-wms-prod-v1"
2021-07-11 17:08:20 [ℹ]  creating Fargate profile "wms-prod-v1" on EKS cluster "cluster-wms-prod-v1"
2021-07-11 17:08:37 [ℹ]  created Fargate profile "wms-prod-v1" on EKS cluster "cluster-wms-prod-v1"
2021-07-11 17:11:07 [ℹ]  "coredns" is now schedulable onto Fargate
2021-07-11 17:13:14 [ℹ]  "coredns" is now scheduled onto Fargate
2021-07-11 17:13:14 [ℹ]  "coredns" pods are now scheduled onto Fargate
2021-07-11 17:15:15 [ℹ]  building iamserviceaccount stack "eksctl-cluster-wms-prod-v1-addon-iamserviceaccount-kube-system-aws-node"
2021-07-11 17:15:15 [ℹ]  deploying stack "eksctl-cluster-wms-prod-v1-addon-iamserviceaccount-kube-system-aws-node"
2021-07-11 17:15:15 [ℹ]  waiting for CloudFormation stack "eksctl-cluster-wms-prod-v1-addon-iamserviceaccount-kube-system-aws-node"
2021-07-11 17:15:32 [ℹ]  waiting for CloudFormation stack "eksctl-cluster-wms-prod-v1-addon-iamserviceaccount-kube-system-aws-node"
2021-07-11 17:15:49 [ℹ]  waiting for CloudFormation stack "eksctl-cluster-wms-prod-v1-addon-iamserviceaccount-kube-system-aws-node"
2021-07-11 17:15:49 [ℹ]  serviceaccount "kube-system/aws-node" already exists
2021-07-11 17:15:49 [ℹ]  updated serviceaccount "kube-system/aws-node"
2021-07-11 17:15:49 [ℹ]  daemonset "kube-system/aws-node" restarted
2021-07-11 17:17:50 [ℹ]  waiting for the control plane availability...
2021-07-11 17:17:50 [✔]  saved kubeconfig as "/home/ubuntu/.kube/config"
2021-07-11 17:17:50 [ℹ]  no tasks
2021-07-11 17:17:50 [✔]  all EKS cluster resources for "cluster-wms-prod-v1" have been created
2021-07-11 17:19:53 [ℹ]  kubectl command should work with "/home/ubuntu/.kube/config", try 'kubectl get nodes'
2021-07-11 17:19:53 [✔]  EKS cluster "cluster-wms-prod-v1" in "me-south-1" region is ready
yiming:~/environment $ 
```

![image-20210712002956454](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210712002956454.png)



验证集群工作状态:

```bash
$ kubectl get nodes
NAME                                                   STATUS   ROLES    AGE   VERSION
fargate-ip-10-163-76-142.me-south-1.compute.internal   Ready    <none>   41m   v1.19.6-eks-e91815
fargate-ip-10-163-76-215.me-south-1.compute.internal   Ready    <none>   41m   v1.19.6-eks-e91815
```

可以看到现在有两个 Fargate 节点在运行，分别承载的是两个 coreDNS Pod。

```bash
$ kubectl get po --all-namespaces -o wide
NAMESPACE     NAME                      READY   STATUS    RESTARTS   AGE   IP              NODE                                                   NOMINATED NODE   READINESS GATES
kube-system   coredns-97b64d8dd-46bf9   1/1     Running   0          42m   10.163.76.142   fargate-ip-10-163-76-142.me-south-1.compute.internal   <none>           <none>
kube-system   coredns-97b64d8dd-hzbs8   1/1     Running   0          42m   10.163.76.215   fargate-ip-10-163-76-215.me-south-1.compute.internal   <none>           <none>
```



查看 eksctl 为我们创建的默认 Fargate Profile，它指定了 default 和 kube-system 两个 namespace 中的 pod 都会运行在 Fargate 上。

```bash
eksctl get fargateprofile --cluster cluster-wms-prod-v1 -o yaml
```

输出的 Fargate Profile 信息示例如下：

```yaml
- name: fp-default
  podExecutionRoleARN: arn:aws:iam::153705321444:role/eksctl-cluster-wms-prod-v1-FargatePodExecutionRole-1NKSYZKHN4L9X
  selectors:
  - namespace: default
  - namespace: kube-system
  status: ACTIVE
  subnets:
  - subnet-09d49c0dc4f8fa13c
  - subnet-0da4b3753f5ff5ecb
- name: wms-prod-v1
  podExecutionRoleARN: arn:aws:iam::153705321444:role/eksctl-cluster-wms-prod-v1-FargatePodExecutionRole-1NKSYZKHN4L9X
  selectors:
  - namespace: wms-prod-v1
  status: ACTIVE
  subnets:
  - subnet-09d49c0dc4f8fa13c
  - subnet-0da4b3753f5ff5ecb
```



至此，我们成功的完成了 Fargate 集群的创建。



