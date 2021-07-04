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



#### 3.2.1、pod的理论

pod里面的容器有两种类型，一种是根容器Pause；一种是用户容器,可以有多个。

根容器的作用：

​       用来评估整个pod内部的容器的健康状态。

​       提供ip给pod内的其他容器共享，用于他们的通讯。（如果是其他pod之间的通讯，用的flannel）



资源清单：就是yaml的配置

```shell
kubectl explain pod   #下面就是资源清单列表能配置的选项
 
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.

FIELDS:
   apiVersion	<string>   # 版本信息，版本号必须是：kubectl api-versions可以查到的
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

   kind	<string>  # 类型，必须是： kubectl api-resources可以查到的
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

   metadata	<Object>   # 资源标识和说明
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

   spec	<Object>   # 对各种资源的描述，是最重要的部分
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

   status	<Object>  # 各种状态信息
     Most recently observed status of the pod. This data may not be up to date.
     Populated by the system. Read-only. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

#上面的是一级属性，如果要看后续的可以一直通过.的方式进行查看
kubectl explain pod.status
```



> *上述的所有配置中，spec是最重要的*

```shell
kubectl  explain pod.spec
KIND:     Pod
VERSION:  v1

RESOURCE: spec <Object>

DESCRIPTION:
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status

     PodSpec is a description of a pod.

FIELDS:
   activeDeadlineSeconds	<integer>
     Optional duration in seconds the pod may be active on the node relative to
     StartTime before the system will actively try to mark it failed and kill
     associated containers. Value must be a positive integer.

   affinity	<Object>
     If specified, the pod's scheduling constraints

   automountServiceAccountToken	<boolean>
     AutomountServiceAccountToken indicates whether a service account token
     should be automatically mounted.

   containers	<[]Object> -required-
     List of containers belonging to the pod. Containers cannot currently be
     added or removed. There must be at least one container in a Pod. Cannot be
     updated.

   dnsConfig	<Object>
     Specifies the DNS parameters of a pod. Parameters specified here will be
     merged to the generated DNS configuration based on DNSPolicy.

   dnsPolicy	<string>
     Set DNS policy for the pod. Defaults to "ClusterFirst". Valid values are
     'ClusterFirstWithHostNet', 'ClusterFirst', 'Default' or 'None'. DNS
     parameters given in DNSConfig will be merged with the policy selected with
     DNSPolicy. To have DNS options set along with hostNetwork, you have to
     specify DNS policy explicitly to 'ClusterFirstWithHostNet'.

   enableServiceLinks	<boolean>
     EnableServiceLinks indicates whether information about services should be
     injected into pod's environment variables, matching the syntax of Docker
     links. Optional: Defaults to true.

   ephemeralContainers	<[]Object>
     List of ephemeral containers run in this pod. Ephemeral containers may be
     run in an existing pod to perform user-initiated actions such as debugging.
     This list cannot be specified when creating a pod, and it cannot be
     modified by updating the pod spec. In order to add an ephemeral container
     to an existing pod, use the pod's ephemeralcontainers subresource. This
     field is alpha-level and is only honored by servers that enable the
     EphemeralContainers feature.

   hostAliases	<[]Object>
     HostAliases is an optional list of hosts and IPs that will be injected into
     the pod's hosts file if specified. This is only valid for non-hostNetwork
     pods.

   hostIPC	<boolean>
     Use the host's ipc namespace. Optional: Default to false.

   hostNetwork	<boolean>
     Host networking requested for this pod. Use the host's network namespace.
     If this option is set, the ports that will be used must be specified.
     Default to false.

   hostPID	<boolean>
     Use the host's pid namespace. Optional: Default to false.

   hostname	<string>
     Specifies the hostname of the Pod If not specified, the pod's hostname will
     be set to a system-defined value.

   imagePullSecrets	<[]Object>
     ImagePullSecrets is an optional list of references to secrets in the same
     namespace to use for pulling any of the images used by this PodSpec. If
     specified, these secrets will be passed to individual puller
     implementations for them to use. For example, in the case of docker, only
     DockerConfig type secrets are honored. More info:
     https://kubernetes.io/docs/concepts/containers/images#specifying-imagepullsecrets-on-a-pod

   initContainers	<[]Object>
     List of initialization containers belonging to the pod. Init containers are
     executed in order prior to containers being started. If any init container
     fails, the pod is considered to have failed and is handled according to its
     restartPolicy. The name for an init container or normal container must be
     unique among all containers. Init containers may not have Lifecycle
     actions, Readiness probes, Liveness probes, or Startup probes. The
     resourceRequirements of an init container are taken into account during
     scheduling by finding the highest request/limit for each resource type, and
     then using the max of of that value or the sum of the normal containers.
     Limits are applied to init containers in a similar fashion. Init containers
     cannot currently be added or removed. Cannot be updated. More info:
     https://kubernetes.io/docs/concepts/workloads/pods/init-containers/

   nodeName	<string>
     NodeName is a request to schedule this pod onto a specific node. If it is
     non-empty, the scheduler simply schedules this pod onto that node, assuming
     that it fits resource requirements.

   nodeSelector	<map[string]string>
     NodeSelector is a selector which must be true for the pod to fit on a node.
     Selector which must match a node's labels for the pod to be scheduled on
     that node. More info:
     https://kubernetes.io/docs/concepts/configuration/assign-pod-node/

   overhead	<map[string]string>
     Overhead represents the resource overhead associated with running a pod for
     a given RuntimeClass. This field will be autopopulated at admission time by
     the RuntimeClass admission controller. If the RuntimeClass admission
     controller is enabled, overhead must not be set in Pod create requests. The
     RuntimeClass admission controller will reject Pod create requests which
     have the overhead already set. If RuntimeClass is configured and selected
     in the PodSpec, Overhead will be set to the value defined in the
     corresponding RuntimeClass, otherwise it will remain unset and treated as
     zero. More info:
     https://git.k8s.io/enhancements/keps/sig-node/20190226-pod-overhead.md This
     field is alpha-level as of Kubernetes v1.16, and is only honored by servers
     that enable the PodOverhead feature.

   preemptionPolicy	<string>
     PreemptionPolicy is the Policy for preempting pods with lower priority. One
     of Never, PreemptLowerPriority. Defaults to PreemptLowerPriority if unset.
     This field is beta-level, gated by the NonPreemptingPriority feature-gate.

   priority	<integer>
     The priority value. Various system components use this field to find the
     priority of the pod. When Priority Admission Controller is enabled, it
     prevents users from setting this field. The admission controller populates
     this field from PriorityClassName. The higher the value, the higher the
     priority.

   priorityClassName	<string>
     If specified, indicates the pod's priority. "system-node-critical" and
     "system-cluster-critical" are two special keywords which indicate the
     highest priorities with the former being the highest priority. Any other
     name must be defined by creating a PriorityClass object with that name. If
     not specified, the pod priority will be default or zero if there is no
     default.

   readinessGates	<[]Object>
     If specified, all readiness gates will be evaluated for pod readiness. A
     pod is ready when all its containers are ready AND all conditions specified
     in the readiness gates have status equal to "True" More info:
     https://git.k8s.io/enhancements/keps/sig-network/0007-pod-ready%2B%2B.md

   restartPolicy	<string>
     Restart policy for all containers within the pod. One of Always, OnFailure,
     Never. Default to Always. More info:
     https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy

   runtimeClassName	<string>
     RuntimeClassName refers to a RuntimeClass object in the node.k8s.io group,
     which should be used to run this pod. If no RuntimeClass resource matches
     the named class, the pod will not be run. If unset or empty, the "legacy"
     RuntimeClass will be used, which is an implicit class with an empty
     definition that uses the default runtime handler. More info:
     https://git.k8s.io/enhancements/keps/sig-node/runtime-class.md This is a
     beta feature as of Kubernetes v1.14.

   schedulerName	<string>
     If specified, the pod will be dispatched by specified scheduler. If not
     specified, the pod will be dispatched by default scheduler.

   securityContext	<Object>
     SecurityContext holds pod-level security attributes and common container
     settings. Optional: Defaults to empty. See type description for default
     values of each field.

   serviceAccount	<string>
     DeprecatedServiceAccount is a depreciated alias for ServiceAccountName.
     Deprecated: Use serviceAccountName instead.

   serviceAccountName	<string>
     ServiceAccountName is the name of the ServiceAccount to use to run this
     pod. More info:
     https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/

   setHostnameAsFQDN	<boolean>
     If true the pod's hostname will be configured as the pod's FQDN, rather
     than the leaf name (the default). In Linux containers, this means setting
     the FQDN in the hostname field of the kernel (the nodename field of struct
     utsname). In Windows containers, this means setting the registry value of
     hostname for the registry key
     HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters to
     FQDN. If a pod does not have FQDN, this has no effect. Default to false.

   shareProcessNamespace	<boolean>
     Share a single process namespace between all of the containers in a pod.
     When this is set containers will be able to view and signal processes from
     other containers in the same pod, and the first process in each container
     will not be assigned PID 1. HostPID and ShareProcessNamespace cannot both
     be set. Optional: Default to false.

   subdomain	<string>
     If specified, the fully qualified Pod hostname will be
     "<hostname>.<subdomain>.<pod namespace>.svc.<cluster domain>". If not
     specified, the pod will not have a domainname at all.

   terminationGracePeriodSeconds	<integer>
     Optional duration in seconds the pod needs to terminate gracefully. May be
     decreased in delete request. Value must be non-negative integer. The value
     zero indicates stop immediately via the kill signal (no opportunity to shut
     down). If this value is nil, the default grace period will be used instead.
     The grace period is the duration in seconds after the processes running in
     the pod are sent a termination signal and the time when the processes are
     forcibly halted with a kill signal. Set this value longer than the expected
     cleanup time for your process. Defaults to 30 seconds.

   tolerations	<[]Object>
     If specified, the pod's tolerations.

   topologySpreadConstraints	<[]Object>
     TopologySpreadConstraints describes how a group of pods ought to spread
     across topology domains. Scheduler will schedule pods in a way which abides
     by the constraints. All topologySpreadConstraints are ANDed.

   volumes	<[]Object>
     List of volumes that can be mounted by containers belonging to the pod.
     More info: https://kubernetes.io/docs/concepts/storage/volumes
```

> containers: 容器列表
>
> nodeName: 根据nodeName来写死pod的调度节点
>
> nodeSelector: 根据标签选择器来选择指定标签的node,然后将pod部署
>
> hostName: 是否使用主机模式，默认false.如果使用主机模式，会将该主机的ip作为pod的ip,但是会出现端口冲突的问题，所以一般不采用
>
> volumes: 存储卷
>
> restartPolicy: 重启策略，当pod出现问题时，重启的策略。



containers:容器列表

```shell
kubectl explain pod.spec.containers  # 查看容器资源配置的信息
KIND:     Pod
VERSION:  v1

RESOURCE: containers <[]Object>

DESCRIPTION:
     List of containers belonging to the pod. Containers cannot currently be
     added or removed. There must be at least one container in a Pod. Cannot be
     updated.

     A single application container that you want to run within a pod.

FIELDS:
   args	<[]string>
     Arguments to the entrypoint. The docker image's CMD is used if this is not
     provided. Variable references $(VAR_NAME) are expanded using the
     container's environment. If a variable cannot be resolved, the reference in
     the input string will be unchanged. The $(VAR_NAME) syntax can be escaped
     with a double $$, ie: $$(VAR_NAME). Escaped references will never be
     expanded, regardless of whether the variable exists or not. Cannot be
     updated. More info:
     https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/#running-a-command-in-a-shell

   command	<[]string>
     Entrypoint array. Not executed within a shell. The docker image's
     ENTRYPOINT is used if this is not provided. Variable references $(VAR_NAME)
     are expanded using the container's environment. If a variable cannot be
     resolved, the reference in the input string will be unchanged. The
     $(VAR_NAME) syntax can be escaped with a double $$, ie: $$(VAR_NAME).
     Escaped references will never be expanded, regardless of whether the
     variable exists or not. Cannot be updated. More info:
     https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/#running-a-command-in-a-shell

   env	<[]Object>
     List of environment variables to set in the container. Cannot be updated.

   envFrom	<[]Object>
     List of sources to populate environment variables in the container. The
     keys defined within a source must be a C_IDENTIFIER. All invalid keys will
     be reported as an event when the container is starting. When a key exists
     in multiple sources, the value associated with the last source will take
     precedence. Values defined by an Env with a duplicate key will take
     precedence. Cannot be updated.

   image	<string>
     Docker image name. More info:
     https://kubernetes.io/docs/concepts/containers/images This field is
     optional to allow higher level config management to default or override
     container images in workload controllers like Deployments and StatefulSets.

   imagePullPolicy	<string>
     Image pull policy. One of Always, Never, IfNotPresent. Defaults to Always
     if :latest tag is specified, or IfNotPresent otherwise. Cannot be updated.
     More info:
     https://kubernetes.io/docs/concepts/containers/images#updating-images

   lifecycle	<Object>
     Actions that the management system should take in response to container
     lifecycle events. Cannot be updated.

   livenessProbe	<Object>
     Periodic probe of container liveness. Container will be restarted if the
     probe fails. Cannot be updated. More info:
     https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle#container-probes

   name	<string> -required-
     Name of the container specified as a DNS_LABEL. Each container in a pod
     must have a unique name (DNS_LABEL). Cannot be updated.

   ports	<[]Object>
     List of ports to expose from the container. Exposing a port here gives the
     system additional information about the network connections a container
     uses, but is primarily informational. Not specifying a port here DOES NOT
     prevent that port from being exposed. Any port which is listening on the
     default "0.0.0.0" address inside a container will be accessible from the
     network. Cannot be updated.

   readinessProbe	<Object>
     Periodic probe of container service readiness. Container will be removed
     from service endpoints if the probe fails. Cannot be updated. More info:
     https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle#container-probes

   resources	<Object>
     Compute Resources required by this container. Cannot be updated. More info:
     https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/

   securityContext	<Object>
     Security options the pod should run with. More info:
     https://kubernetes.io/docs/concepts/policy/security-context/ More info:
     https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

   startupProbe	<Object>
     StartupProbe indicates that the Pod has successfully initialized. If
     specified, no other probes are executed until this completes successfully.
     If this probe fails, the Pod will be restarted, just as if the
     livenessProbe failed. This can be used to provide different probe
     parameters at the beginning of a Pod's lifecycle, when it might take a long
     time to load data or warm a cache, than during steady-state operation. This
     cannot be updated. More info:
     https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle#container-probes

   stdin	<boolean>
     Whether this container should allocate a buffer for stdin in the container
     runtime. If this is not set, reads from stdin in the container will always
     result in EOF. Default is false.

   stdinOnce	<boolean>
     Whether the container runtime should close the stdin channel after it has
     been opened by a single attach. When stdin is true the stdin stream will
     remain open across multiple attach sessions. If stdinOnce is set to true,
     stdin is opened on container start, is empty until the first client
     attaches to stdin, and then remains open and accepts data until the client
     disconnects, at which time stdin is closed and remains closed until the
     container is restarted. If this flag is false, a container processes that
     reads from stdin will never receive an EOF. Default is false

   terminationMessagePath	<string>
     Optional: Path at which the file to which the container's termination
     message will be written is mounted into the container's filesystem. Message
     written is intended to be brief final status, such as an assertion failure
     message. Will be truncated by the node if greater than 4096 bytes. The
     total message length across all containers will be limited to 12kb.
     Defaults to /dev/termination-log. Cannot be updated.

   terminationMessagePolicy	<string>
     Indicate how the termination message should be populated. File will use the
     contents of terminationMessagePath to populate the container status message
     on both success and failure. FallbackToLogsOnError will use the last chunk
     of container log output if the termination message file is empty and the
     container exited with an error. The log output is limited to 2048 bytes or
     80 lines, whichever is smaller. Defaults to File. Cannot be updated.

   tty	<boolean>
     Whether this container should allocate a TTY for itself, also requires
     'stdin' to be true. Default is false.

   volumeDevices	<[]Object>
     volumeDevices is the list of block devices to be used by the container.

   volumeMounts	<[]Object>
     Pod volumes to mount into the container's filesystem. Cannot be updated.

   workingDir	<string>
     Container's working directory. If not specified, the container runtime's
     default will be used, which might be configured in the container image.
     Cannot be updated.
```



> 样例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dev-nginx
  namespace: dev
  labels:
    type: oddworld
spec:
  containers:
  - name:  nginx
    image: nginx
  - name: busybox
    image: busybox
```

查看对用的启动情况：

```shell
kubectl get pod -n dev  # 查看启动的情况，发现busy box 出现：Back-off restarting failed container，而且可以发现已经重启了5次了。
NAME                                READY   STATUS             RESTARTS   AGE
dev-nginx                           1/2     CrashLoopBackOff   5          20s
nginx-deployment-66b6c48dd5-57v5f   1/1     Running            1          23h
nginx-deployment-66b6c48dd5-d8vc6   1/1     Running            1          23h
nginx-deployment-66b6c48dd5-wq84f   1/1     Running            1          23h

[root@k8s-01 pod]# kubectl describe pod dev-nginx -n dev
Name:         dev-nginx
Namespace:    dev
Priority:     0
Node:         k8s-03/172.29.98.72
Start Time:   Sat, 03 Jul 2021 11:06:09 +0800
Labels:       type=oddworld
Annotations:  <none>
Status:       Running
IP:           10.244.2.12
IPs:
  IP:  10.244.2.12
Containers:
  nginx:
    Container ID:   docker://4e5a2c31d7ae76f6cd12545a715025bc1f0a7179eef3c8ad457e713043a7658f
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:47ae43cdfc7064d28800bc42e79a429540c7c80168e8c8952778c0d5af1c09db
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Sat, 03 Jul 2021 11:06:13 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-bl2lk (ro)
  busybox:
    Container ID:   docker://4466ff2c55055d89505926872f832ee43802cce2f739e7b3a57d35b53e5d2ee1
    Image:          busybox
    Image ID:       docker-pullable://busybox@sha256:930490f97e5b921535c153e0e7110d251134cc4b72bbb8133c6a5065cc68580d
    Port:           <none>
    Host Port:      <none>
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Sat, 03 Jul 2021 11:06:44 +0800
      Finished:     Sat, 03 Jul 2021 11:06:44 +0800
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Sat, 03 Jul 2021 11:06:25 +0800
      Finished:     Sat, 03 Jul 2021 11:06:25 +0800
    Ready:          False
    Restart Count:  2
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-bl2lk (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  kube-api-access-bl2lk:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age               From               Message
  ----     ------     ----              ----               -------
  Normal   Scheduled  39s               default-scheduler  Successfully assigned dev/dev-nginx to k8s-03
  Normal   Pulling    39s               kubelet            Pulling image "nginx"
  Normal   Pulled     35s               kubelet            Successfully pulled image "nginx" in 3.620273117s
  Normal   Created    35s               kubelet            Created container nginx
  Normal   Started    35s               kubelet            Started container nginx
  Normal   Pulled     26s               kubelet            Successfully pulled image "busybox" in 8.452693171s
  Normal   Pulled     23s               kubelet            Successfully pulled image "busybox" in 3.122599227s
  Normal   Pulling    8s (x3 over 35s)  kubelet            Pulling image "busybox"
  Normal   Created    5s (x3 over 26s)  kubelet            Created container busybox
  Normal   Pulled     5s                kubelet            Successfully pulled image "busybox" in 3.329484301s
  Normal   Started    4s (x3 over 26s)  kubelet            Started container busybox
  Warning  BackOff    4s (x3 over 22s)  kubelet            Back-off restarting failed container
```

>镜像的拉取策略：
>
>imagePullPolicy:  
>
>​    Always: 总是去远程拉取
>
>​    IfNotPresent:  本地有用本地，本地没有去远程仓库拉取
>
>​    Never: 只使用本地的
>
>默认值说明：
>
>​    如果使用的是具体的版本号，那么就是使用IfNotPresent
>
>​    如果使用的lastest，或者没有指定版本号，那么就是使用Always



有一些软件，启动后，如果没有执行相关命名，k8s会主动关闭掉，那么如果需要一直运行，就需要使用到命令的参数配置: Command



比如上述的busybox这个软件，就是这种情况的， 如果不加入一些让它一直执行的命令，它会主动关闭掉对应的容器。

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dev-nginx
  namespace: dev
  labels:
    type: oddworld
spec:
  containers:
  - name:  nginx
    image: nginx
  - name: busybox
    image: busybox 
    command: ["/bin/sh","-c","touch /tmp/hello.txt; while true; do /bin/echo $(date + %T) >> /tmp/hello.txt; sleep 3; done"]
    
# 通过启动： kubectl apply -f dev.yaml
# kubectl apply -f dev.yamll
NAME                                READY   STATUS    RESTARTS   AGE
dev-nginx                           2/2     Running   0          15s
发现两个都已经启动了，然后k8需要进去看下，对应的命令是否已经开启成功
# kubectl exec dev-nginx -n dev -it -c busybox /bin/sh
进入对应的环境之后，就需要进行查看对应的文件是否已经创建
tail -f /tmp/hello.txt
```

环境变量：env其实是一个键值对的数组。配置的环境变量，会替换springboot的系统变量，这个优先级比较高。

```shell
apiVersion: v1
kind: Pod
metadata:
  name: dev-nginx
  namespace: dev
  labels:
    type: oddworld
spec:
  containers:
  - name:  nginx
    image: nginx
  - name: busybox
    image: busybox 
    command: ["/bin/sh","-c","touch /tmp/hello.txt; while true; do /bin/echo $(date + %T) >> /tmp/hello.txt; sleep 3; done"]
    env:
    - name: jeffchan
      value: caraliu


kubectl exec dev-nginx -n dev -it -c busybox /bin/sh  # 进入容器

echo $jeffchan
caraliu

```



设置端口号：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dev-nginx
  namespace: dev
  labels:
    type: oddworld
spec:
  containers:
  - name:  nginx
    image: nginx
  - name: busybox
    image: busybox 
    command: ["/bin/sh","-c","touch /tmp/hello.txt; while true; do /bin/echo $(date + %T) >> /tmp/hello.txt; sleep 3; done"]
    env:
    - name: jeffchan
      value: caraliu
    ports:
    - name: busy-port
      containerPort: 800
      protocol: TCP #默认就是TCP,只能是TCP,UDP,sctp
```

**资源配置**：

>如果不对资源做相关的限制，那么某个容器可能会因为某些异常，吃掉大量的内存，从而导致其他容器受到影响，所以需要通过资源的限制来防止这种问题的发生。
>
>limits: 限制容器的最大资源，当超过这个设置的值之后，容器会被重启
>
>requests : 这只资源的最小值，如果小于这个值，那么容器将启动不起来 

```
apiVersion: v1
kind: Pod
metadata:
  name: dev-nginx
  namespace: dev
  labels:
    type: oddworld
spec:
  containers:
  - name:  nginx
    image: nginx
  - name: busybox
    image: busybox 
    command: ["/bin/sh","-c","touch /tmp/hello.txt; while true; do /bin/echo $(date + %T) >> /tmp/hello.txt; sleep 3; done"]
    env:
    - name: jeffchan
      value: caraliu
    ports:
    - name: busy-port
      containerPort: 800
      protocol: TCP #默认就是TCP,只能是TCP,UDP,sctp
    resources:
      limits:
        cpu: "2"
        memory: "10Gi"
      requests:
        cpu: "1"
        memory: "10Gi"
        
 cpu: 核心数，可以是小数
 memory: 内存大小，G,M,Gi，Mi
```



#### 3.2.2、pod的生命周期

**pod的生命周期就是指从容器创建到终止的整个过程**

>创建pod
>
>运行初始化容器
>
>运行主容器
>
>​     容器启动后的钩子，容器关闭前钩子
>
>​     容器的存活性探测，容器的就绪性探测
>
>pod的终止过程

**整个过程中会出现五种状态**（相位）

> 挂起：apiServer已经创建了pod,尚未调度完成或者还处在下载镜像的过程中
>
> 运行中：已经调度了对应的节点，同时所有的容器已经创建成功
>
> 成功：容器成功执行完，不会被重启，比如busybox启动后执行一个命令
>
> 失败：所有容器都已经终止：但至少有一个容器返回了非0的退出状态。
>
> 未知：通常是网络导致的，apiServer无法获取到pod的状态信息



**创建pod的流程**

>用户通过kubelet或者其他的api客户端提交需要创建pod的信息给apiServer
>
>apiServer开始生成pod对象的信息，并将信息存入etcd中，然后返回确认给客户端
>
>apiServer开始反映etcd的pod对象的变化，其他组件通过watch机制来检查apiServer的变动
>
>scheduler发现有新的pod对象需要创建，开始给pod分配主机并将结果信息更新到apiServer
>
>node接待你上的kubelet发现有pod调度过来，尝试调用docker启动容器，并将结果返回给apiServer
>
>apiServer将接收到的pod的状态存入到etcd中

**终止pod的过程**

> 用户想apiServer发送删除pod对象的命令
>
> apiServer
>
> 将pod标记为terminating状态
>
> 



**运行初始化容器**

> **特征**:
>
> 初始化容器必须完成成功，不成功一直重试
>
> 初始化容器按照顺序执行
>
> 
>
> **运用场景**：
>
> 提供容器组不具备的工具程序或者自定义程序
>
> 初始化容器要先于运用容器完成，因此可以做一些该运行容器的一些依赖处理。



```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dev-nginx
  namespace: dev
  labels:
    type: oddworld
spec:
  containers:
  - name:  nginx
    image: nginx
  - name: busybox
    image: busybox 
    command: ["/bin/sh","-c","touch /tmp/hello.txt; while true; do /bin/echo $(date + %T) >> /tmp/hello.txt; sleep 3; done"]
    env:
    - name: jeffchan
      value: caraliu
    ports:
    - name: busy-port
      containerPort: 800
      protocol: TCP #默认就是TCP,只能是TCP,UDP,sctp
    resources:
      limits:
        cpu: "2"
        memory: "10Gi"
      requests:
        cpu: "1"
        memory: "10Gi"
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
```



**钩子函数**

>pre start: 在容器启动后执行
>
>pre stop: 在容器结束关闭前执行

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dev-nginx
  namespace: dev
  labels:
    type: oddworld
spec:
  containers:
  - name:  nginx
    image: nginx
  - name: busybox
    image: busybox 
    command: ["/bin/sh","-c","touch /tmp/hello.txt; while true; do /bin/echo $(date + %T) >> /tmp/hello.txt; sleep 3; done"]
    env:
    - name: jeffchan
      value: caraliu
    ports:
    - name: busy-port
      containerPort: 800
      protocol: TCP #默认就是TCP,只能是TCP,UDP,sctp
    resources:
      limits:
        cpu: "2"
        memory: "10Gi"
      requests:
        cpu: "1"
        memory: "10Gi"
    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh","-c", "echo postStart > /hsr/share/nginx/html/index.html"]
      preStop:
        exec:
          command: ["/usr/sbin/nginx", "-s", "quit"]
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
```



**容器探测**

> 容器探测是来监控出异常的实例，将它剔除，不承担业务流量
>
> 
>
> **有两种探针**：
>
>  liveness probes: 存活性探针，如果探测实例是否处于正常运行状态，如果不是，k8s会重启容器
>
>  readiness probes:  就绪性探针，用于检查实例是否可以接受请求，如果不是，k8s不会转发流量
>
> 
>
> **三种探测方式**：
>
> Exec:
>
> 
>
> TCPSocket:
>
> 
>
> HTTPGet:



**重启策略**：失败后，会延时10s,20,40,60...重启

>Always: 容器失效时及重启，默认值
>
>Onfailure:  容器终止运行且不为零0时重启
>
>Never: 不论什么状态都不重启



#### 3.2.3、pod的调度

pod的调度是由scheduler来控制的，scheduler采用某些调度算法进行计算，然后计算出调度的节点，最后pod就在该节点创建运行。

> 自动调度：流行在哪个节点完全由scheduler经过一系列算法计算得出
>
> 定向调度：NodeName  NodeSelector
>
> 亲和性调度：NodeAffinity  PodAffniity  PodAntiAffnity
>
> 污点（容忍）调度：Taints  Toleration, 指node由污点，容忍是指pod能容忍多少个。

 

**定向调度**：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dev-nginx
  namespace: dev
  labels:
    type: oddworld
spec:
  containers:
  - name:  nginx
    image: nginx
  - name: busybox
    image: busybox 
    command: ["/bin/sh","-c","touch /tmp/hello.txt; while true; do /bin/echo $(date + %T) >> /tmp/hello.txt; sleep 3; done"]
    env:
    - name: jeffchan
      value: caraliu
    ports:
    - name: busy-port
      containerPort: 800
      protocol: TCP #默认就是TCP,只能是TCP,UDP,sctp
    resources:
      limits:
        cpu: "2"
        memory: "10Gi"
      requests:
        cpu: "1"
        memory: "10Gi"
    lifecycle:
      postStart:
        exec:
          command: ["/bin/sh","-c", "echo postStart > /hsr/share/nginx/html/index.html"]
      preStop:
        exec:
          command: ["/usr/sbin/nginx", "-s", "quit"]
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  nodeName: node1
  nodeSelector:
     run: dev
```



**亲和度调度**：

>nodeAffinity
>
>   ```yaml
># 强制的节点亲和性调度
>apiVersion: v1
>kind: Pod
>metadata:
>  name: nginx
>spec:
>  affinity:
>    nodeAffinity:
>      requiredDuringSchedulingIgnoredDuringExecution:
>        nodeSelectorTerms:
>        - matchExpressions:
>          - key: disktype
>            operator: In
>            values:
>            - ssd            
>  containers:
>  - name: nginx
>    image: nginx
>    imagePullPolicy: IfNotPresent
>    
>---
># 使用首选的节点亲和性调度
>apiVersion: v1
>kind: Pod
>metadata:
>  name: nginx
>spec:
>  affinity:
>    nodeAffinity:
>      preferredDuringSchedulingIgnoredDuringExecution:
>      - weight: 1
>        preference:
>          matchExpressions:
>          - key: disktype
>            operator: In
>            values:
>            - ssd          
>  containers:
>  - name: nginx
>    image: nginx
>    imagePullPolicy: IfNotPresent
>   ```
>
>
>
>podAffinity
>
>
>
>podAntiAffintity







### 3.3、Label



### 3.4、Deployment



### 3.5、Service

首先创建服务：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: dev
spec:
  replicas: 3
  selector: 
    mathLable:
      run: nginx
  template:
    metadata:
      labels:
        run: nginx
      spec:
        containers:
        - image: nginx
          ports:
          - containerPort: 80
            protocol: TCP
```



pod会因为pod自身出现问题会重启或者重新创建然后导致ip变化，那么对应外部访问就不太友好。

pod还因为用的是集群内部的ip，所以不能为外部访问。

此时要通过service做一层处理，然后service和控制器的标签关联，然后关联到控制器控制管理的pod。

那么就可以通过范文service来访问pod。同时还提供了负载均衡的能力。

将上述部署到deployment控制器的几个pod，通过service暴露出去：

```shell
kubectl get deploy -n dev
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           11m
kubectl expose deploy nginx-deployment --name=svc-nginx --type=ClusterIP --port=80 --target-port=80 -n dev
service/svc-nginx exposed  #可以看到服务已经暴露

kubectl get service -n dev -o wide   #查看对应的service
NAME        TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE    SELECTOR
svc-nginx   ClusterIP   10.97.88.48   <none>        80/TCP    2m9s   app=nginx

不过这种只是暴露到集群内部，如果要暴露到集群外部，那么需要将--type改成NodePort的方式

kubectl expose deploy nginx-deployment --name=svc-nginx2 --type=NodePort --port=80 --target-port=80 -n dev
service/svc-nginx2 exposed

然后可以通过:
[root@k8s-01 deployment]# kubectl get service -n dev -o wide
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE    SELECTOR
svc-nginx    ClusterIP   10.97.88.48     <none>        80/TCP         5m3s   app=nginx
svc-nginx2   NodePort    10.99.236.236   <none>        80:31491/TCP   41s    app=nginx
```



查看外部访问：外网IP:31491

![image-20210702115415593](D:\my-learning\oddworld-learn-note\k8s\k8s.assets\image-20210702115415593.png)

```shell
kubectl delete service svc-nginx -n dev #删除服务
service "svc-nginx" deleted
```

通过配置文件的方式：svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
   name: svc-nginx
   namespace: dev
spec:
   ports:
   - port: 80
     protocol: TCP
     targetPort: 80
   selector:
       run: nginx
   type: ClusterIP
```

然后执行：

```yaml
kubectl  apply -f  svc.yaml  

kubectl get service -n dev #查看dev命名空间内的service

NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
svc-nginx    ClusterIP   10.98.39.83     <none>        80/TCP         7s
svc-nginx2   NodePort    10.99.236.236   <none>        80:31491/TCP   3h8m
```



