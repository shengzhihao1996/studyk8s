01-Devops核心要点及kubernetes架构概述
02-kubernetes基础概念
03-kubeadm初始化Kubernetes集群（注明：此方法只适用于学习和测试）
04-kubernetes应用快速入门
05-kubernetes资源清单定义入门
06-Kubernetes Pod控制器应用进阶
07-Kubernetes Pod控制器应用进阶
08-Kubernetes Pod控制器
09-Kubernetes Pod控制器
10-kubernetes Service资源
11-kubernetes ingress及Ingress Controller
12-存储卷
14-kubernetes statefulset控制器
15-kubernetes认证及serviceaccount
16-kubernetes RBAC
17-kubernetes dashboard认证及分级授权
22-容器资源需求、资源限制及HeapSter
附件：
所有模版均在github上备份，各位看官可去猫猫
01-Devops核心要点及kubernetes架构概述
1.容器编排工具：
docker compose 、docker swarm ,docker machine
mesos,marathon
kubernetes（80%市场份额）

DevOps应用模式的开发

批量管理应用程序：（多场景、多模式）
1、构建同一版本的不同配置的应用程序
2、上传本地仓库
3、修改yml文件
4、apply
（呕~~~~~~~缺点自己想吧！）

使用configmap，修改yml文件中容器启动时加载的configmap，实现一个初始应用镜像 多场景、多模式启动，便于管理

02-kubernetes基础概念
master/node
  master:api-server  scheduler  controller-manager
  node: kubelet docker kube-proxy
pod  label  label selector
  label:key=value
  label selector:
  namespace：管理边界
pod
  自主式pod:api-server接受创建请求，交由scheduler调度器将pod创建指令调度至指定节点，由节点启动pod。容器和pod的售后由kubectl完成，but节点故障=game over。
  控制器管理的pod：有生命周期的对象，由调度器将其调度至集群中的某节点。
    replicationcontroller：
    replicaset：
    deployment：无状态  :hpa 水平pod自动伸缩控制器  
    statefulset：有状态
    deamset：one node one pod
    job、ctonjob:作业
    
hpa:
  horizontalpodautoscaler: 对容器资源指标监控

addons:附件
用户请求到达物理服务器地址——转发代理至service——代理转发至nginx——代理转发至Tomcat——到mysql

network
  节点——集群——pod
  同一pod内容器通信：lo
  pod通信： 
  pod与service通信：
  
 cni:容器网络接口
  flannel：网络配置
  calico：网络配置，网络策略
  canel：要flannel的简单网络配置，要calico的网络策略功能（结合体）


03-kubeadm初始化Kubernetes集群（注明：此方法只适用于学习和测试）


1、本地解析
2、kube和docker安装（点击链接）
3、从国外拉取镜像（推荐github+阿里云私有仓库）（点击链接）
4、编写docker的json，harbor和镜像加速
cat >/etc/docker/daemon.json<<e
{
  "registry-mirrors": ["https://5ynvh8bq.mirror.aliyuncs.com"],
  "insecure-registries": ["registry"]
}
e
systemctl daemon-reload 
systemctl restart docker
5、禁用swap，开启内核转发
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl -p /etc/sysctl.d/k8s.conf
swapoff -a && sysctl -w vm.swappiness=0
去除swap开机挂载
6、开机自启docker和kubelet
systemctl enable docker.service kubelet.service 
7、初始化master给定网段和版本
kubeadm init --kubernetes-version=v1.12.2  --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12

root执行前两条即可
8、安装网络插件
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

04-kubernetes应用快速入门
kubectl get 查找模块及其简写
通过coredns解析service，从而在内部节点访问service后端的pod服务



kubectl expose deployment nginx-deploy  --name nginx --port=80 --target-port=80 --protocol=TCP

kubectl scale --replicas=3 deployment myapp 直接操作deployment，控制pod数量
kubectl set image deployment myapp myapp=ikubernetes/myapp:v2
kubectl rollout undo deployment myapp


clusterip——》nodeport

05-kubernetes资源清单定义入门
apiVersion: group/version      #kubectl api-versions
kind:                          #资源类别
metadata:                      #元数据
  name:
  namespace:
  labels:
  annotations:
#资源引用path：/api/GROUP/VERSION/namespaces/NAMESPACE/TYPE/NAME         #替换大写内容
spec:                          #用户期望的目标状态
status：                       #当前状态

root@master1:~/data/k8s/lesson-5# cat pod-demo.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
  namespace: default
  labels:
    app: myapp
    tier: frontend
spec:
  containers:
    - name: myapp 
      image: ikubernetes/myapp:v1
    - name: busybox
      image: busybox:latest
      command:
        - "/bin/sh"
        - "-c"
        - "sleep 3600"

06-Kubernetes Pod控制器应用进阶
 标签选择
root@master1:~# kubectl get pods -L app
NAME       READY     STATUS    RESTARTS   AGE       APP
pod-demo   2/2       Running   1          1h        myapp
root@master1:~# kubectl get pods -L myapp
NAME       READY     STATUS    RESTARTS   AGE       MYAPP
pod-demo   2/2       Running   1          1h        
root@master1:~# kubectl get pods -L app=myapp
NAME       READY     STATUS    RESTARTS   AGE       APP=MYAPP
pod-demo   2/2       Running   1          1h        

root@master1:~# kubectl get pods -l app
NAME       READY     STATUS    RESTARTS   AGE
pod-demo   2/2       Running   1          1h
root@master1:~# kubectl get pods -l myapp
No resources found.
root@master1:~# kubectl get pods -l app=myapp
NAME       READY     STATUS    RESTARTS   AGE
pod-demo   2/2       Running   1          1h
 查询 -l, --selector='': Selector (label query) to filter on, supports '=', '==', and '!='.(e.g. -l key1=value1,key2=value2)
列表  -L, --label-columns=[]: Accepts a comma separated list of labels that are going to be presented as columns. Names are case-sensitive. You can also use multiple flag options like -L label1 -L label2...
 kubectl label nodes node1 disktype=ssd
liveness probe
readness probe
restartPolicy: always,onfailure,never(max:300s)
 
root@master1:~/data/k8s/lesson-6# cat pod-demo.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
  namespace: default
  labels:
    app: myapp
    tier: frontend
  annotations:
    szh/create-by: "cluster admin"
spec:
  containers:
    - name: myapp 
      image: ikubernetes/myapp:v1
      ports:
      - name: http
        containerPort: 80
      - name: https
        containerPort: 443 
    - name: busybox
      image: busybox:latest
      command:
      - "/bin/sh"
      - "-c"
      - "sleep 3600"
  nodeSelector:
    disktype: ssd

 
07-Kubernetes Pod控制器应用进阶
检测机制：
exec——文件
httpget——请求
tcpsocket——不解释
liveness-exec-pod.yaml
apiVersion: v1
kind: Pod 
metadata: 
  name: liveness-exec-pod
  namespace: default
spec:
  containers:
  - name: liveness-exec-container
    image: busybox:latest
    command: ["/bin/sh","-c","touch /tmp/healthy;sleep 30;rm -rf /tmp/healthy;sleep 3600"]
    livenessProbe:
      exec:
        command: ["test","-e","/tmp/healthy"]
      initialDelaySeconds: 1
      periodSeconds: 3
      

liveness-httpget-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-httpget-container
  namespace: default
spec:
  containers:
  - name: liveness-httpget-container
    image: ikubernetes/myapp:v1
    imagePullPolicy: IfNotPresent
    ports: 
    - name: http
      containerPort: 80
    livenessProbe:
      httpGet:
        port: http
        path: /index.html
      initialDelaySeconds: 1
      periodSeconds: 3

readiness-httpget-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-httpget-pod
  namespace: default
spec:
  containers:
  - name: readiness-httpget-pod
    image: ikubernetes/myapp:v1
    imagePullPolicy: IfNotPresent
    ports:
    - name: http
      containerPort: 80
    readinessProbe:
      httpGet:
        port: http
        path: /index.html
      initialDelaySeconds: 1
      periodSeconds: 3

08-Kubernetes Pod控制器
replicationController:
replicaSet:
deployment:无状态
daemonSet: 一个节点，一个pod 
job：
cronjob：
statefulSet:有状态
灰度：逐个滚动更新
金丝雀：只更新一个
蓝绿：逐批滚动更新

root@master1:~/data/k8s/lesson-8# cat rs-demo.yaml 
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
      release: canary
  template:
    metadata:
      name: myapp
      labels:
        app: myapp
        release: canary
        environment: qa
    spec:
      containers:
      - name: myapp-container
        image: ikubernetes/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 80

09-Kubernetes Pod控制器
#补丁修改rs
kubectl patch deployment myapp-deploy -p '{"spec":{"replicas":5}}'

kubectl patch deployment myapp-deploy -p '{"spec":{"strategy":{"rollingUpdate":{"maxSurge":1,"maxUnavailable":0}}}}'

kubectl set image deployment myapp-deploy myapp=ikubernetes/myapp:v3

  kubectl rollout resume deployment myapp-deploy
  kubectl rollout history deployment myapp-deploy
  kubectl rollout undo  deployment myapp-deploy --to-revision=2


myapp-deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deploy
  namespace: default
spec:
  replicas: 4
  selector:
    matchLabels:
      app: myapp
      release: canary
  template:
    metadata:
      labels:
        app: myapp
        release: canary
    spec:
      containers:
      - name: myapp
        image: ikubernetes/myapp:v1
        imagePullPolicy: IfNotPresent
        ports:
        - name: http
          containerPort: 80

myapp-ds.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
      role: logstor
  template:
    metadata:
      labels:
        app: redis
        role: logstor
    spec:
      containers:
      - name: redis
        image: redis:4.0-alpine
        ports:
        - name: redis
          containerPort: 6379
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: filebeat-ds 
  namespace: default
spec:
  selector:
    matchLabels:
      app: filebeat
      release: stable
  template:
    metadata:
      labels:
        app: filebeat
        release: stable
    spec:
      containers:
      - name: filebeat
        image: ikubernetes/filebeat:5.6.5-alpine
        env:
        - name: REDIS_HOST
          value: redis.default.svc.cluster.local
        - name: REDIS_LOG_LEVEL
          value: info

10-kubernetes Service资源
域名：svc.name.ns_name.domain.ltd.
root@master1:~/data/k8s/lesson-10# cat redis-svc.yaml 
apiVersion: v1
kind: Service
metadata:
  name: myapp
  namespace: default
spec:
  selector:
    app: myapp
    release: canary
  clusterIP: 10.254.97.97
  type: NodePort
  ports: 
  - port: 80
    targetPort: 80
    nodePort: 30080


无头服务：
root@master1:~/data/k8s/lesson-10# cat redis-headless.yaml 
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc
  namespace: default
spec:
  selector:
    app: myapp
    release: canary
  clusterIP: None
  ports: 
  - port: 80
    targetPort: 80



11-kubernetes ingress及Ingress Controller

首先，ssl加密认证是未来必不可少的，在集群内部部署加密业务就是此次的课题
用户通过node节点至svc负载调度至后端pod，理论上后端pod全部配置ssl可以实现加密访问
but， svc属于四层负载，不存在卸载会话的功能，所以用户每访问一个pod就要实现一次会话ssl，会严重拖累访问速度。
这里引入ingress，后端pod不用配置ssl，只在ingress配置，提高用户体验





将ingress-nginx以pod+host的形式架在node上，负载后端pod
利用service抓取伸缩的pod地址池


 mandatory.yaml  #建立ingress-pod，动态监听svc的地址池
 service-nodeport.yaml  #给ingress-pod建立一个svc，实现负载的任务
 ingress-myapp.yaml  #域名测试
 test-myapp.yaml #tcp80测试pod

加密：
openssl genrsa -out tls.key 2048
openssl req -new -x509 -key tls.key -out tls.crt -subj /C=CN/ST=Beijing/O=Devops/CN=tomcat.com
kubectl create secret tls tomcat-ingress-secret --cert=tls.crt --key=tls.key
root@szh:~/test/mysql/ssl# kubectl create secret tls tomcat-ingress-secret --cert=cert-1540981351989_nxyc.shengzhihao.top.crt  --key=cert-1540981351989_nxyc.shengzhihao.top.key

部署ingress负载
mandatory.yaml
service-nodeport.yaml
课题一：
ingress-nginx负载三个80/tcp
ingress-myapp.yaml
test-myapp.yaml
课题二：
ingress-nginx负载三个8080/tcp
ingress-tomcat.yaml
test-tomcat.yaml
课题三：
ingress-nginx负载三个8080/tcp（ssl）
ingress-tomcat-ssl.yaml
test-tomcat.yaml
ssl/tls.crt
ssl/tls.key

12-存储卷
1、在pod上定义volume
2、在container中使用volumemounts

1、pod内共享存储（打通mount namespace）
volume-pod-emptyDir.yaml
2、节点持久化存储（pod必须打死在某一节点上）
volume-pod-hostPath.yaml
3、nfs持久化存储
volume-pod-nfs.yaml
4、固定的pv、pvc
perdistentvolume.yaml
test-pvc.yaml
5、pvc动态请求pv

6、configmap创建
root@master1:~/data/k8s/lesson-12# kubectl create configmap nginx-www --from-file=./www.conf  ^C
root@master1:~/data/k8s/lesson-12# kubectl describe cm
Name:         nginx-www
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
www.conf:
----
server {
   server_name szh.com;
   listen 80;
   root   /data/web/html/;
 }

Events:  <none>
root@master1:~/data/k8s/lesson-12# kubectl edit  cm

# Please edit the object below. Lines beginning with a '#' will be ignored,
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  www.conf: "server {\n \tserver_name szh.com;\n \tlisten 80;\n \troot   /data/web/html/;\n
    }\n"
kind: ConfigMap
metadata:
  creationTimestamp: 2018-10-28T10:58:38Z
  name: nginx-www
  namespace: default
  resourceVersion: "263840"
  selfLink: /api/v1/namespaces/default/configmaps/nginx-www
  uid: 6c533f0e-daa0-11e8-b0c4-000c29e987bc


使用仓库认证信息：(harbor仓库)
root@master1:~/data/k8s/lesson-12# kubectl create secret 
Create a secret using specified subcommand.

Available Commands:
  docker-registry Create a secret for use with a Docker registry #用户认证镜像仓库
  generic         Create a secret from a local file, directory or literal value
  tls             Create a TLS secret  #证书私钥

root@master1:~# kubectl explain po.spec.imagePullSecrets.name
KIND:     Pod
VERSION:  v1

FIELD:    name <string>

DESCRIPTION:
     Name of the referent. More info:
     https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#names

将secret以环境变量的形式注入容器
kubectl create secret generic mysql-root-password --from-literal=password=MYSQL123456
root@master1:~/data/k8s/lesson-12# cat test-secret.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret
  namespace: default
  labels:
    app: myapp
    tier: frontend
  annotations:
    szh.com/create-by: "cluster admin"
spec:
  containers:
  - name: myapp
    image: ikubernetes/myapp:v1
    ports:
    - name: http
      containerPort: 80
    env:
    - name: MYSQL-ROOT-PASSWORD
      valueFrom:
        secretKeyRef:
          name: mysql-root-password
          key: password

14-kubernetes statefulset控制器
statefulset：
1、稳定且唯一的网络标识符。
2、稳定且持久的存储。
3、有序、平滑的部署、扩展。
4、有序、平滑的删除、终止。
5、有序的滚动更新

1、headless service
2、StatefulSet
3、volumeClaimTemplate  #卷申请模版

域名解析：
pod_name.service_name.ns_name.svc.cluster.local

nfs：
apt-get install nfs-kernel-server
apt-get install nfs-common

扩容、缩容：
kubectl scale sts myapp --replicas=5
kubectl patch sts myapp -p '{"spec":{"replicas":2}}'
kubectl set image sts/myapp myapp=ikubernetes/myapp:v2

15-kubernetes认证及serviceaccount
用户通过访问认证（令牌、token、ssl（确认服务器身份）） 
用户通过授权检查（RBAC基于角色的访问控制）
准入控制（涉及到级联授权）

客户端——》API server：
    user：user_name,user_id
    group: 
    extra:

  API:
     Request path
        hppt://ip:6443/apis/apps/v1/namespaces/default/deployments/myapp-deploy/
      HTTP request verb(请求动作)
         get post put delete  
       api request verb
           get list create update patch  watch  proxy  redirct  delete deletecollection(一组一个集合)
       resource:
       subresource
       namespace
       apigroup    
  
api<---->  认证  svc  kubernetes   
kubectl客户端
pod客户端

kubectl create serviceaccount admin

root@master1:~/data/k8s/lesson-15# cat pod-demo.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod-demo
  namespace: default
  labels:
    app: myapp
    tier: frontend
  annotations:
    szh/create-by: "cluster admin"
spec:
  containers:
  - name: myapp 
    image: ikubernetes/myapp:v1
    ports:
    - name: http
      containerPort: 80
  serviceAccountName: admin

root@master1:~/data/k8s/lesson-15# kubectl describe sa admin
Name:                admin
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   admin-token-s2z65
Tokens:              admin-token-s2z65
Events:              <none>

创建证书：
(umask 077 ;openssl genrsa -out szh.key 2048)
openssl req -new -key szh.key -out szh.csr -subj "/CN=szh"
openssl x509 -req -in szh.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out szh.crt -days 365

  set-credentials Sets a user entry in kubeconfig：

kubectl config set-credentials szh --client-certificate=./szh.crt --client-key=./szh.key --embed-certs=true

  set-context     Sets a context entry in kubeconfig：


kubectl config set-context szh@kubernetes --cluster=kubernetes   --user=szh

 
 use-context     Sets the current-context in a kubeconfig file：

kubectl config use-context szh@kubernetes
kubectl config set-cluster   szh --kubeconfig=/tmp/test.txt --server="https://192.168.1.61:6443" --certificate-authority=/etc/kubernetes/pki/ca.crt --embed-certs=true
kubectl config view --kubeconfig=/tmp/test.txt
  
#kubectl proxy --port=8080利用kubectl认证apiserver，然后curl
#rest 表针状态转移，承载对象状态信息数据格式，序列化数据结构json yaml
上下文contexts：一个 kubectl控制多集群

16-kubernetes RBAC

基于角色的授权RBAC Role-based AC
一切皆对象
资源：集群、名称空间两个级别
kubectl create role pods-reader --verb=get,list,watch --resource=pods
在default namespaces下给pods资源授予get list watch  权限
kubectl create rolebinding szh-read-pods --role=pods-reader --user=szh
将role绑定到szh

rolebinding ---->role  |clusterrole  ---->某user有对所划分namespace下的某资源有某权限
clusterrolebinding ---->clusterrole  ----->某user有对所划分集群下的某资源有某权限

kubectl get serviceaccount -n kube-system
kubectl get role --all-namespaces
kubectl get rolebinding   --all-namespaces

RBAC:
  role,rolebinding
  clusterrole,clusterrolebinding

rolebinding,clusterrolebinding:
  subject:
    user
    group
    serviceaccount

role,clusterrole:
  object:
    resource group
    resource
    non-resource url
  action:
    get,list,watch,patch,delete,deletecollection
    

17-kubernetes dashboard认证及分级授权
向dashboard提供的认证文件必须是serviceaccount 

管理员令牌：
  kubectl create serviceaccount dashboard-admin -n kube-system
  kubectl create clusterrolebinding dashboard-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
  kubectl get secret -n kube-system |grep dashboard
  kubectl describe  secret dashboard-admin-token-899f6 -n kube-system

default namespace用户的令牌：
  kubectl create serviceaccount def-ns-admin -n default
  kubectl create rolebinding def-ns-admin --clusterrole=admin --serviceaccount=default:def-ns-admin
  kubectl get secret |grep def-ns-admin
   kubectl  describe secret def-ns-admin-token-s9rz7




22-容器资源需求、资源限制及HeapSter
CPU: 
     1=1000,millicores
        500m=0.5CPU 







~                                                                                                                                                                                                                                                                             

附件：
apiserver要抓取kubelet，获取其信息
标签不能乱用，一旦控制器和pod的标签绑定了，会出现误操作现象



kubelet周期性的更新自身状态到apiserver,周期为–node-status-update-frequency 默认值10s
controller manager每–-node-monitor-period会检查kubelet上报上来的状态，默认值为5s
node的状态将会–node-monitor-grace-period周期内更新，controller manager中设置默认值为40s
kubelet在更新自身状态的时候将会有nodeStatusUpdateRetry 次重试，当前nodeStatusUpdateRetry 设置的默认值为5次 
所以将会发生nodeStatusUpdateRetry * –node-status-update-frequency才能更新一次node的状态。同时，controller manager将会每个–node-monitor-period周期内检查nodeStatusUpdateRetry 次，当–node-monitor-grace-period周期之后将会认为node unhealthy. 它将会根据–pod-eviction-timeout时间来移除pods
--------------------- 
作者：阿仆来耶 
来源：CSDN 
原文：https://blog.csdn.net/jettery/article/details/81121774
echo "source <(kubectl completion bash)" >> ~/.bashrc   自动补全
source <(kubectl completion zsh)
yum install bash-completion -y


      --pid string                     PID namespace to use
      --uts string                     UTS namespace to use
      --ipc string                     IPC mode to use
      --network string                 Connect a container to a network (default "default")
      --mount mount                    Attach a filesystem mount to the container
      --user string                    Username or UID (format: <name|uid>[:<group|gid>])


  kubeadm join 172.16.1.10:6443 --token gf1a9u.2lugy15lybe6mwyz --discovery-token-ca-cert-hash sha256:24db87ed1bede74619b2c3df8af255d5cfde1fd92b7ce5508b2c091e4ae08142

