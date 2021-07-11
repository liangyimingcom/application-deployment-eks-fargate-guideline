# AWS VPC环境规划 - VPC的设计与部署



**提示：如果您在任何步骤创建失败，环境清理请参考 [步骤5]()。**



## 第一步：对要部署的应用，进行VPC规划设计

我们以部署在AWS巴林区域的一个名称为“SMW”的应用为例。



**VPC CIDR规划：: 10.163.76.0/23 (508)**

**VPC子网规划 - Subnet- EKS/k8s/worknode cluster  -** 

|      | **Description**         | **Subnets**      | **Type** | **memo** |
| ---- | ----------------------- | ---------------- | -------- | :------: |
|      |                         |                  |          |          |
| 1    | External  subnet - AZ a | 10.163.76.0/26   | public   |    62    |
| 2    | External  subnet - AZ b | 10.163.76.64/26  | public   |    62    |
|      |                         |                  |          |          |
| 3    | Internal  subnet - AZ a | 10.163.76.128/26 | private  |    62    |
| 4    | Internal  subnet - AZ b | 10.163.76.192/26 | private  |    62    |
|      |                         |                  |          |          |

 **VPC子网规划 -  Subnet - RDS/ElasticCache**    

|      | **Description**         | **Subnets**     | **Type** | **memo** |
| ---- | ----------------------- | --------------- | -------- | -------- |
|      |                         |                 |          |          |
| 1    | Internal subnet - AZ  a | 10.163.77.0/27  | private  | 31       |
| 2    | Internal subnet - AZ  b | 10.163.77.32/27 | private  | 31       |

**VPC子网规划说明：EKS自动创建VPC分区，k8s worknodes分为二类子网**

- 外网 (External subnet 1&2)： 公共子网
- 内网 (Internal subnet 1&2)： 私有子网



![image-20210712000641894](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210712000641894.png)





## 第二步：对规划VPC，进行部署，同时完成fargateEKS的标签要求



使用Cloudformation 创建 WMS PROD VPC
1）打开巴林的CF：https://me-south-1.console.aws.amazon.com/cloudformation/home?region=me-south-1#/

2）创建新的stacks，填入：https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml

![image-20210712001115457](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210712001115457.png)



3）创建新的stacks name = WMS-PROD-v1，填入IP地址（参考 VPC规划设计）

![image-20210712001243406](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210712001243406.png)

![image-20210712001254886](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210712001254886.png)



4）确认无误后，点击创建CF。然后检查VPC是否创建成功。

![image-20210712001313036](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210712001313036.png)



5）创建RDS/REDIS子网；（图例仅供参考）

![image-20210712001347851](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210712001347851.png)



6）检查VPC是否正常，如下图：（图例仅供参考）

![image-20210712001411569](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210712001411569.png)



## 第三步：准备 fargateEKS集群的Yaml配置文件

**Yaml文件名：《fargate-existing-vpc-wms-prod-v1.yaml》** 。在Cloud9里面”New Terminal“，然后复制粘贴如下代码创建文件。

```yaml
cat << EOF > fargate-existing-vpc-wms-prod-v1.yaml
--- 
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: cluster-wms-prod-v1
  region: me-south-1

vpc:
  id: "vpc-0f62b217700e212ab"  # (需要修改，必须填入正确的VPC ID)
  cidr: "10.163.76.0/23"       
  subnets:
    public:
      public-one:
        cidr: "10.163.76.0/26"
      public-two:
        cidr: "10.163.76.64/26"
    private:
      private-one:
        cidr: "10.163.76.128/26"
      private-two:
        cidr: "10.163.76.192/26"

fargateProfiles:
  - name: fp-default
    selectors:
      # All workloads in the "default" Kubernetes namespace will be
      # scheduled onto Fargate:
      - namespace: default
      # All workloads in the "kube-system" Kubernetes namespace will be
      # scheduled onto Fargate:
      - namespace: kube-system
  - name: wms-prod-v1
    selectors:
      # All workloads in the "wms" Kubernetes namespace will be scheduled onto Fargate:
      - namespace: wms-prod-v1

iam:
  withOIDC: true
EOF
```

![image-20210712005117301](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210712005117301.png)



1) 在Cloud9中打开yaml文件，然后修改VPC-ID，如AWS Console一致：

![image-20210712005239684](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210712005239684.png)



2）VPC-ID来自于AWS Console的VPC界面，请在这里查阅：

![image-20210712005320596](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210712005320596.png)



3）请把改好的YAML文件保存好，用于下一步的操作。



**注意：使用Cloudformation 创建 WMS PROD VPC的主要目的是为了完成VPC打标签（Tag）的工作。没有正确配置标签的VPC子网（subnet）是无法在fargateEKS下工作的。**

正确的标签参考如下：

![image-20210712002629206](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210712002629206.png)



![image-20210712002654179](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210712002654179.png)

