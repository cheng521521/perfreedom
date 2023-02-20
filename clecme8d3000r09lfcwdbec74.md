# EKS上传日志到CloudWatch

# 准备工作

## 创建OIDC IAM

使用如下命令

`<cluster_name>` 是你EKS集群的名字

```c
aws eks describe-cluster --name <cluster_name> --query "cluster.identity.oidc.issuer" --output text
```

1. 打开AWS控制台,进入IAM服务
    

[https://console.aws.amazon.com/iam/](https://console.aws.amazon.com/iam/)

1. 在导航窗格中，选择 \*\*Identity Providers (身份提供商)，\*\*然后选择 **Create Provider (创建提供商)**
    
    1. 对于 **Provider Type（提供商类型）**，选择 **Choose a provider type（选择提供商类型）**，然后选择 **OpenID Connect**。
        
    2. 对于 **Provider URL（提供商 URL）**，粘贴集群的 OIDC 发布者 URL。
        
    3. 对于受众，键入 **sts.amazonaws.com** 并选择 **Next Step (下一步)**。
        

3.验证提供商信息正确，然后选择 \*\*Create（创建）\*\*以创建身份提供商。

## 创建IAM role和policy

1. 选择自定义信任策略
    

占位符讲解

* $account\_id 账户ID,登录控制台的时候就有
    
* $oidc\_provider 执行 `aws eks describe-cluster --name <cluster_name> --query "cluster.identity.oidc.issuer" --output text` 输出的内容
    
* $namespace 集群的命名空间
    
* $service\_account 集群的sa账户,这里可以写\*
    

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::$account_id:oidc-provider/$oidc_provider"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "$oidc_provider:aud": "sts.amazonaws.com",
          "$oidc_provider:sub": "system:serviceaccount:$namespace:$service_account"
        }
      }
    }
  ]
}
```

# 创建名为amazon-cloudwatch的命名空间

```bash
kubectl create ns amazon-cloudwatch
```

或者直接执行下面

```bash
kubectl apply -f [https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cloudwatch-namespace.yaml](https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cloudwatch-namespace.yaml)
```

# 创建ConfigMap

将 my-cluster-name 和 my-cluster-region 替换为您的集群名称和区域。

```bash
ClusterName=<my-cluster-name>
RegionName=<my-cluster-region>
FluentBitHttpPort='2020'
FluentBitReadFromHead='Off'
[[ ${FluentBitReadFromHead} = 'On' ]] && FluentBitReadFromTail='Off'|| FluentBitReadFromTail='On'
[[ -z ${FluentBitHttpPort} ]] && FluentBitHttpServer='Off' || FluentBitHttpServer='On'
kubectl create configmap fluent-bit-cluster-info \
--from-literal=cluster.name=${ClusterName} \
--from-literal=http.server=${FluentBitHttpServer} \
--from-literal=http.port=${FluentBitHttpPort} \
--from-literal=read.head=${FluentBitReadFromHead} \
--from-literal=read.tail=${FluentBitReadFromTail} \
--from-literal=logs.region=${RegionName} -n amazon-cloudwatch
```

# 将 Fluent Bit 优化配置 DaemonSet 部署到该集群

```bash
kubectl apply -f [https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/fluent-bit/fluent-bit.yaml](https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/fluent-bit/fluent-bit.yaml)
```

# 将K8S的SA账户关联到IAM上

```bash
kubectl annotate serviceaccounts fluent-bit -n amazon-cloudwatch "[eks.amazonaws.com/role-arn=arn:aws:iam::ACCOUNT_ID:role/IAM_ROLE_NAME](http://eks.amazonaws.com/role-arn=arn:aws:iam::ACCOUNT_ID:role/IAM_ROLE_NAME)"
```

# **删除 Fluent Bit 部署**

```bash
kubectl delete -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/fluent-bit/fluent-bit.yaml
```

## 如何调试

打开 `amazon-cloudwatch` 命令空间下的ConfigMap`fluent-bit-config` ,把其中的`log-level`等级改为 `debug`.使用如下命令查看日志信息

```bash
kubectl logs <pod name>  -n <namespace> | head -n 500
```

如果出现如下错误,那就是IAM没有配置好，要仔仔细细的配置

```bash
[aws_credentials] sts assume role request failed
```