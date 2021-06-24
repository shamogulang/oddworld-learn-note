<center><h1>k8s</h1></center>
## 1、集群的安装

官网的安装教程：https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/



记得对应的环境要加：

```
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF
```

然后重启docker



```
kubeadm join 172.29.98.73:6443 --token abcdef.0123456789abcdef \
	--discovery-token-ca-cert-hash sha256:939006a5c2c8ab3d9d6b3aec2cc18cbba915d0233bea952597cc91b177b08395
```



## 2、kubectl命令

2.1、资源管理方式

k8s的资源管理方式分为命令式和文件式的管理方式

命令方式：

```shell
kubectl version  #查看版本

kubectl cluster-info  #查看集群信息

kubectl get node  #查看节点信息

kubectl create namespace dev #创建一个命名空间dev

kubectl delete namespace dev #删除一个命名空间

kubectl get ns  [命名空间名称] #获取命名空间

kubectl get pod  --all-namespace #查看所有的pod信息

kubectl get pod [podName] #查看指定的pod的信息

kubectl get pod podName] [-o wide/yaml] #查看比较完整的信息，wide查看的比较多一点信息，如果用yaml的话，信息会非常详细

kubectl run nginx-name --image=nginx -n dev #运行nginx镜像，然后对应的pod的名称为：nginx-name

kubectl describe pod ngnix [--namespace=dev] #查看一个Pod的详细信息

kubectl logs nginx  --namespace=dev #查看dev命名空间下的nginx的日志情况

kubectl delete pod nginx -n dev #删除一个pod，但是它会重建
```



文件方式：

```shell
kubectl create -f  pod.yaml # 根据文件的内容，创建一个pod

kubectl patch -f pod.yaml # 根据文件的内容，修改一个pod

kubectl apply -f pod.yaml # 根据文件的内容，创建或修改一个pod
```

  配置文件内容；vim pod.yaml

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
  
---
apiVersion: v1
kind: Pod
metadata:
  name: nginx-meta-yaml
  namespace: dev
spec:
  containers:
  - name: ngnix-yaml
    image: niginx:1.17.1
```

如果想在其他节点也使用kubectl 命名，那么需要：

scp -r  ~/.kube  nodehostname:  ~/

