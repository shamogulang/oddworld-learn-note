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



## 2、DashBoard

获取对应的dashboard的yaml

wget  https://raw.githubusercontent.com/kubernetes/dashboard/v2.2.0/aio/deploy/recommended.yaml

打开文件，修改其中几个地方

```yaml
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort  #这个时加的
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30009  #这个时加的
  selector:
    k8s-app: kubernetes-dashboard
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



## 3、核心的内容

### 3.1、namespace

用于进行资源的隔离：如果不想要pod相互访问，就可以建立多个命名空间。

多租户管理：加上授权机制，那么不同的用户看到的pod就不一样，操作的资源不一样。



```shell
kubectl get namespace  # 获取所有的namespace
NAME              STATUS   AGE
default           Active   7m45s   没有指定命名空间的pod会放到这里
kube-node-lease   Active   7m46s   用于心跳检测，集群节点的心跳维护
kube-public       Active   7m46s   所有人都可以访问，不需要授权即可访问
kube-system       Active   7m46s   k8s本身的资源放到这里

[root@k8s-01 flanel]# kubectl get pods -n kube-system
NAME                             READY   STATUS    RESTARTS   AGE
coredns-558bd4d5db-mpz8v         1/1     Running   0          11m
coredns-558bd4d5db-pslpr         1/1     Running   0          11m
etcd-k8s-01                      1/1     Running   0          11m
kube-apiserver-k8s-01            1/1     Running   0          11m
kube-controller-manager-k8s-01   1/1     Running   0          11m
kube-flannel-ds-mz2st            1/1     Running   0          9m35s
kube-flannel-ds-rqs68            1/1     Running   0          9m35s
kube-flannel-ds-z4pkp            1/1     Running   0          9m35s
kube-proxy-fj7z9                 1/1     Running   0          11m
kube-proxy-rpvdd                 1/1     Running   0          11m
kube-proxy-sjpm2                 1/1     Running   0          11m
kube-scheduler-k8s-01            1/1     Running   0          11m

[root@k8s-01 flanel]# kubectl describe namespace default
Name:         default
Labels:       kubernetes.io/metadata.name=default
Annotations:  <none>
Status:       Active

No resource quota.  #这里是针对namespace做限制的描述，这里可以看到没有做限制

No LimitRange resource. # 这里是针对组件的资源做限制，这里也是没有做任何的限制

kubectl create ns dev  #创建命名空间
namespace/dev created
kubectl delete ns dev  #删除命名空间
namespace "dev" deleted
```



通过配置文件的方式进行创建namespace,创建文件dev.yaml

```yaml
apiVersion: v1.0
kind: Namspace
metadata:
  name: dev
```

然后可以执行：

```shell
kubectl create -f dev.yaml  #创建命名空间

kubectl delete -f dev.yaml  #删除命名空间
```



### 3.2、Pod

用于对容器进行封装，一个pod里面可以放一个或者多个容器。pod是k8s的最小的单位。

其实k8s的一些组件也是通过pod来运行的。比如下面的：

```shell
[root@k8s-01 flanel]# kubectl get pod --namespace=kube-system
NAME                             READY   STATUS    RESTARTS   AGE
coredns-558bd4d5db-mpz8v         1/1     Running   0          38m  # 用作dns
coredns-558bd4d5db-pslpr         1/1     Running   0          38m  # 用作dns
etcd-k8s-01                      1/1     Running   0          38m  # 用作分布式存储
kube-apiserver-k8s-01            1/1     Running   0          38m  # 用户请求的转发
kube-controller-manager-k8s-01   1/1     Running   0          38m  # 
kube-flannel-ds-mz2st            1/1     Running   0          36m  # 用作网络的构建，每个节点都要跑一个
kube-flannel-ds-rqs68            1/1     Running   0          36m
kube-flannel-ds-z4pkp            1/1     Running   0          36m
kube-proxy-fj7z9                 1/1     Running   0          38m  # 用作代理，每个节点都有一个
kube-proxy-rpvdd                 1/1     Running   0          38m
kube-proxy-sjpm2                 1/1     Running   0          38m
kube-scheduler-k8s-01            1/1     Running   0          38m  # 用作调度，选择将对应的服务部署到哪个节点
```



创建一个pod:

```shell
[root@k8s-01 flanel]# kubectl run dev-nginx --image=nginx --port=80 --namespace=dev
pod/dev-nginx created
如果后面的--namespace=dev不加的话，它会默认放到default的命名空间去。
```



查看pod的信息：

```shell
[root@k8s-01 flanel]# kubectl get pod --namespace=dev -o wide
NAME        READY   STATUS    RESTARTS   AGE     IP           NODE     NOMINATED NODE   READINESS GATES
dev-nginx   1/1     Running   0          3m21s   10.244.2.2   k8s-03   <none>           <none>

可以通过以下命名访问下nginx看下是否正常工作
[root@k8s-01 flanel]# curl 10.244.2.2:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```



查看pod的详情：kubectl describe pod dev-nginx --namespace=dev

```shell
[root@k8s-01 flanel]# kubectl describe pod dev-nginx --namespace=dev
Name:         dev-nginx
Namespace:    dev
Priority:     0
Node:         k8s-03/172.29.98.72
Start Time:   Sun, 27 Jun 2021 15:40:22 +0800
Labels:       run=dev-nginx
Annotations:  <none>
Status:       Running
IP:           10.244.2.2
IPs:
  IP:  10.244.2.2
Containers:
  dev-nginx:
    Container ID:   docker://5a09c47d7d14b8a2ab396d99a98b33a7a6f65738afd596dfb5317eb813fc43dc
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:47ae43cdfc7064d28800bc42e79a429540c7c80168e8c8952778c0d5af1c09db
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Sun, 27 Jun 2021 15:40:36 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>   #表示没有定义环境变量
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zn7q2 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-zn7q2:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:  # 这里可以用于排错
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  9m12s  default-scheduler  Successfully assigned dev/dev-nginx to k8s-03
  Normal  Pulling    9m12s  kubelet            Pulling image "nginx"
  Normal  Pulled     8m58s  kubelet            Successfully pulled image "nginx" in 13.960170431s
  Normal  Created    8m58s  kubelet            Created container dev-nginx
  Normal  Started    8m58s  kubelet            Started container dev-nginx
```



查看pod的日志信息：可以用于查看pod失败的一些日志。

```
[root@k8s-01 flanel]# kubectl logs dev-nginx -n dev
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2021/06/27 07:40:36 [notice] 1#1: using the "epoll" event method
2021/06/27 07:40:36 [notice] 1#1: nginx/1.21.0
2021/06/27 07:40:36 [notice] 1#1: built by gcc 8.3.0 (Debian 8.3.0-6) 
2021/06/27 07:40:36 [notice] 1#1: OS: Linux 3.10.0-957.21.3.el7.x86_64
2021/06/27 07:40:36 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2021/06/27 07:40:36 [notice] 1#1: start worker processes
2021/06/27 07:40:36 [notice] 1#1: start worker process 31
2021/06/27 07:40:36 [notice] 1#1: start worker process 32
10.244.0.0 - - [27/Jun/2021:07:44:30 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"
10.244.0.0 - - [27/Jun/2021:07:44:46 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.29.0" "-"
```



删除pod: kubectl delete pod  dev-nginx -n dev 

```
[root@k8s-01 flanel]# kubectl delete pod dev-nginx -n dev
pod "dev-nginx" deleted
[root@k8s-01 flanel]# kubectl get pod -n dev
No resources found in dev namespace.

这里可以看到删除成功了，去查看也没有了，但是很多时候，你会发现你删除了，pod控制器会马上创建一个新的pod给你。
我们这里能删除成功的原因是：
kubectl get deployment -n dev
No resources found in dev namespace.  我们这里是没有pod控制器的，所以就能删除。
```







### 3.3、Label



### 3.4、Deployment



### 3.5、Service

