# 步骤3 - 部署 AWS LoadBalancer Controller



[AWS Load Balancer Controller](https://github.com/kubernetes-sigs/aws-load-balancer-controller) 是一个控制器，可帮助管理 Kubernetes 集群的 Elastic Load Balancer。

- 它通过部署和配置 Application Load Balancers - ALB 来提供 Kubernetes Ingress 资源。
- 它通过部署和配置 Network Load Balancers - NLB 来提供 Kubernetes Service 资源。



AWS ALB 和 NLB 可以和部署在 Fargate 上的 Service/Ingress 进行集成，通过 IP 模式将流量从负载均衡器直接转发到 Pod IP 上。其中，

- 对于 Kubernetes Service，可以使用 AWS 网络负载均衡器 (NLB，IP 模式) 对跨 Pod 的 4 层网络流量进行负载均衡。
- 对于 Kubernetes Ingress，可以使用 AWS 应用负载均衡器 (ALB，IP 模式) 对跨 Pod 的 7 层网络流量进行负载均衡。



参考资料：https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html

### 创建 IAM 策略

下载 IAM 策略文件，用于为 AWS Load Balancer Controller 配置 [IRSA](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) 权限 

```bash
curl -o iam_policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json
```

创建成功后输出示例如下：

```bash
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  7273  100  7273    0     0  19240      0 --:--:-- --:--:-- --:--:-- 19240
```



使用下载到策略文件创建 IAM 策略

执行如下命令，注意将REGION_NAME区域名称替换成你的应用所在区域值。

```bash
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy-<REGION_NAME> \
    --policy-document file://iam_policy.json
```

*举例：以巴林部署SMW为例子，配置文件代码如下*

```bash
$ aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy-bahrain \
    --policy-document file://iam_policy.json
{
    "Policy": {
        "PolicyName": "AWSLoadBalancerControllerIAMPolicy-bahrain",
        "PolicyId": "ANPASHSMNCPSJE337SKB7",
        "Arn": "arn:aws:iam::153705321444:policy/AWSLoadBalancerControllerIAMPolicy-bahrain",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2021-07-11T17:55:32+00:00",
        "UpdateDate": "2021-07-11T17:55:32+00:00"
    }
}
```





### 创建 IAM role 和 service account

为 AWS Load Balancer Controller 创建一个 IAM role 和 Service Account 并将两者进行关联，以便 AWS Load Balancer Controller 所在 Pod 拥有相应的 IAM 权限（如创建 ELB、TargetGroup 等）。

执行如下命令，注意将 cluster 名称和 policy-arn 替换成你自己的值。

```bash
eksctl create iamserviceaccount \
  --cluster=<CLUSTER_NAME> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --override-existing-serviceaccounts \
  --approve
```

*举例：以巴林部署SMW为例子，配置文件代码如下*

```bash
eksctl create iamserviceaccount \
  --cluster=cluster-wms-prod-v1 \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::153705321444:policy/AWSLoadBalancerControllerIAMPolicy-bahrain \
  --override-existing-serviceaccounts \
  --approve
```



创建成功后 eksctl 输出示例如下：

```bash
2021-07-11 17:56:07 [ℹ]  eksctl version 0.55.0
2021-07-11 17:56:07 [ℹ]  using region me-south-1
2021-07-11 17:56:08 [ℹ]  1 existing iamserviceaccount(s) (kube-system/aws-node) will be excluded
2021-07-11 17:56:08 [ℹ]  1 iamserviceaccount (kube-system/aws-load-balancer-controller) was included (based on the include/exclude rules)
2021-07-11 17:56:08 [!]  metadata of serviceaccounts that exist in Kubernetes will be updated, as --override-existing-serviceaccounts was set
2021-07-11 17:56:08 [ℹ]  1 task: { 2 sequential sub-tasks: { create IAM role for serviceaccount "kube-system/aws-load-balancer-controller", create serviceaccount "kube-system/aws-load-balancer-controller" } }
2021-07-11 17:56:08 [ℹ]  building iamserviceaccount stack "eksctl-cluster-wms-prod-v1-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"
2021-07-11 17:56:08 [ℹ]  deploying stack "eksctl-cluster-wms-prod-v1-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"
2021-07-11 17:56:08 [ℹ]  waiting for CloudFormation stack "eksctl-cluster-wms-prod-v1-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"
2021-07-11 17:56:24 [ℹ]  waiting for CloudFormation stack "eksctl-cluster-wms-prod-v1-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"
2021-07-11 17:56:41 [ℹ]  waiting for CloudFormation stack "eksctl-cluster-wms-prod-v1-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"
2021-07-11 17:56:41 [ℹ]  created serviceaccount "kube-system/aws-load-balancer-controller"
```

创建完成后，可以开始部署 AWS Load Balancer Controller。



### 部署 AWS Load Balancer Controller

可以通过 Helm 或者手工的方式部署，在本次实验中我们使用 Helm 的方式。

#### 安装 Helm

```bash
curl -sSL https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

验证 Helm 安装

```bash
$ helm version --short
v3.6.2+gee407bd
```



#### 安装 Controller

首先安装 TargetGroupBinding 这个 CRD

```bash
kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller//crds?ref=master"
```

添加 eks-chart

```bash
helm repo add eks https://aws.github.io/eks-charts
```

使用 Helm 安装 Controller，将 cluster-name 替换成你的集群名称，region-name 替换成你的 EKS 集群所在 region，vpc-id 替换成你的 EKS 集群所在 VPC ID

```bash
helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller \
  --set clusterName=eks-fargate \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region-name> \
  --set vpcId=<vpc-id> \
  -n kube-system
```

*举例：以巴林部署SMW为例子，配置文件代码如下*

```bash
$ helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller \
  --set clusterName=cluster-wms-prod-v1 \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=me-south-1\
  --set vpcId=vpc-0f62b217700e212ab\
  -n kube-system
Release "aws-load-balancer-controller" does not exist. Installing it now.
NAME: aws-load-balancer-controller
LAST DEPLOYED: Sun Jul 11 18:00:14 2021
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
AWS Load Balancer controller installed!
```



可以使用以下命令获取 VPC ID

```bash
aws ec2 describe-vpcs --filter Name=cidr,Values=10.163.76.0/23 | jq -r '.Vpcs[0].VpcId'
```



查看部署状态和日志

```bash
$ kubectl get deployment -n kube-system aws-load-balancer-controller
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
aws-load-balancer-controller   0/2     2            0           40s
```


查看 Controller pod 名称

```bash
$ kubectl get po  -n kube-system | grep load-balancer
aws-load-balancer-controller-6868948648-6p2qp   0/1     ContainerCreating   0          97s
aws-load-balancer-controller-6868948648-nqzz4   0/1     ContainerCreating   0          97s
```

查看 Controller 日志，注意将 pod name 替换成你自己的 Controller pod 名称

```bash
$ kubectl logs -n kube-system aws-load-balancer-controller-6868948648-6p2qp
{"level":"info","ts":1626026523.5794103,"msg":"version","GitVersion":"v2.2.1","GitCommit":"27803e3f8e3b637873f9bb59c56b78de01f65b79","BuildDate":"2021-06-25T17:18:28+0000"}
{"level":"info","ts":1626026523.68036,"logger":"controller-runtime.metrics","msg":"metrics server is starting to listen","addr":":8080"}
{"level":"info","ts":1626026523.7735102,"logger":"setup","msg":"adding health check for controller"}
{"level":"info","ts":1626026523.7740898,"logger":"controller-runtime.webhook","msg":"registering webhook","path":"/mutate-v1-pod"}
{"level":"info","ts":1626026523.7742438,"logger":"controller-runtime.webhook","msg":"registering webhook","path":"/mutate-elbv2-k8s-aws-v1beta1-targetgroupbinding"}
{"level":"info","ts":1626026523.7743614,"logger":"controller-runtime.webhook","msg":"registering webhook","path":"/validate-elbv2-k8s-aws-v1beta1-targetgroupbinding"}
{"level":"info","ts":1626026523.7744935,"logger":"controller-runtime.webhook","msg":"registering webhook","path":"/validate-networking-v1beta1-ingress"}
{"level":"info","ts":1626026523.7747188,"logger":"setup","msg":"starting podInfo repo"}
{"level":"info","ts":1626026525.7749126,"logger":"controller-runtime.manager","msg":"starting metrics server","path":"/metrics"}
I0711 18:02:05.775941       1 leaderelection.go:242] attempting to acquire leader lease  kube-system/aws-load-balancer-controller-leader...
I0711 18:02:05.792705       1 leaderelection.go:252] successfully acquired lease kube-system/aws-load-balancer-controller-leader
{"level":"info","ts":1626026525.8751626,"logger":"controller-runtime.webhook.webhooks","msg":"starting webhook server"}
{"level":"info","ts":1626026525.8753638,"logger":"controller","msg":"Starting EventSource","reconcilerGroup":"elbv2.k8s.aws","reconcilerKind":"TargetGroupBinding","controller":"targetGroupBinding","source":"kind source: /, Kind="}
{"level":"info","ts":1626026525.875451,"logger":"controller","msg":"Starting EventSource","reconcilerGroup":"elbv2.k8s.aws","reconcilerKind":"TargetGroupBinding","controller":"targetGroupBinding","source":"kind source: /, Kind="}
{"level":"info","ts":1626026525.875478,"logger":"controller","msg":"Starting EventSource","reconcilerGroup":"elbv2.k8s.aws","reconcilerKind":"TargetGroupBinding","controller":"targetGroupBinding","source":"kind source: /, Kind="}
{"level":"info","ts":1626026525.8753161,"logger":"controller","msg":"Starting EventSource","controller":"service","source":"kind source: /, Kind="}
{"level":"info","ts":1626026525.8756387,"logger":"controller","msg":"Starting Controller","controller":"service"}
{"level":"info","ts":1626026525.875251,"logger":"controller","msg":"Starting EventSource","controller":"ingress","source":"channel source: 0xc0004fddb0"}
{"level":"info","ts":1626026525.876165,"logger":"controller","msg":"Starting EventSource","controller":"ingress","source":"channel source: 0xc0004fde00"}
{"level":"info","ts":1626026525.8761897,"logger":"controller","msg":"Starting EventSource","controller":"ingress","source":"kind source: /, Kind="}
{"level":"info","ts":1626026525.8762136,"logger":"controller","msg":"Starting EventSource","controller":"ingress","source":"kind source: /, Kind="}
{"level":"info","ts":1626026525.8762624,"logger":"controller","msg":"Starting EventSource","controller":"ingress","source":"kind source: /, Kind="}
{"level":"info","ts":1626026525.876483,"logger":"controller-runtime.certwatcher","msg":"Updated current TLS certificate"}
{"level":"info","ts":1626026525.876678,"logger":"controller-runtime.webhook","msg":"serving webhook server","host":"","port":9443}
{"level":"info","ts":1626026525.876838,"logger":"controller-runtime.certwatcher","msg":"Starting certificate watcher"}
{"level":"info","ts":1626026525.9757025,"logger":"controller","msg":"Starting workers","controller":"service","worker count":3}
{"level":"info","ts":1626026525.9759219,"logger":"controller","msg":"Starting EventSource","reconcilerGroup":"elbv2.k8s.aws","reconcilerKind":"TargetGroupBinding","controller":"targetGroupBinding","source":"kind source: /, Kind="}
{"level":"info","ts":1626026525.9764557,"logger":"controller","msg":"Starting EventSource","controller":"ingress","source":"channel source: 0xc0004fde50"}
{"level":"info","ts":1626026525.976607,"logger":"controller","msg":"Starting EventSource","controller":"ingress","source":"kind source: /, Kind="}
{"level":"info","ts":1626026526.076321,"logger":"controller","msg":"Starting Controller","reconcilerGroup":"elbv2.k8s.aws","reconcilerKind":"TargetGroupBinding","controller":"targetGroupBinding"}
{"level":"info","ts":1626026526.0765712,"logger":"controller","msg":"Starting workers","reconcilerGroup":"elbv2.k8s.aws","reconcilerKind":"TargetGroupBinding","controller":"targetGroupBinding","worker count":3}
{"level":"info","ts":1626026526.0768263,"logger":"controller","msg":"Starting EventSource","controller":"ingress","source":"kind source: /, Kind="}
{"level":"info","ts":1626026526.0768542,"logger":"controller","msg":"Starting Controller","controller":"ingress"}
{"level":"info","ts":1626026526.0769176,"logger":"controller","msg":"Starting workers","controller":"ingress","worker count":3}

```









