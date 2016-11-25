# kube
由于不能从google container上直接pull镜像，所以这里通过docker hub的Automated Builds功能从项目的dockerfile中Build到docker的官方服务器上，然后再从它们上面拉取.

##	kube 1.4需要的镜像:
```
gcr.io/google_containers/kube-proxy-amd64                v1.4.6
gcr.io/google_containers/kube-discovery-amd64            1.0
gcr.io/google_containers/kubedns-amd64                   1.9
gcr.io/google_containers/kube-scheduler-amd64            v1.4.6
gcr.io/google_containers/kube-controller-manager-amd64   v1.4.6
gcr.io/google_containers/kube-apiserver-amd64            v1.4.6
gcr.io/google_containers/etcd-amd64                      2.3.7
gcr.io/google_containers/kube-dnsmasq-amd64              1.4
gcr.io/google_containers/exechealthz-amd64               1.2
gcr.io/google_containers/pause-amd64                     3.0
kubernetes/heapster                                      canary
gcr.io/google_containers/kubernetes-dashboard-amd64      v1.4.2
```

## docker hub上设置
由于docker hub不能后期更改一个image的tag，所以每次更新kubernetes时，都在build settings中，手动增加一个版本对应文件

## 更改tag
```
images=(kube-proxy-amd64:v1.4.6 kube-discovery-amd64:1.0 kubedns-amd64:1.9 kube-scheduler-amd64:v1.4.6 kube-controller-manager-amd64:v1.4.6 kube-apiserver-amd64:v1.4.6 etcd-amd64:2.3.7 kube-dnsmasq-amd64:1.4 exechealthz-amd64:1.2 pause-amd64:3.0)
for imageName in ${images[@]} ; do
  docker pull ist0ne/$imageName
  docker tag ist0ne/$imageName gcr.io/google_containers/$imageName
done

images=(heapster:canary heapster_grafana:v3.1.1 heapster_influxdb:v0.7)
for imageName in ${images[@]} ; do
  docker pull ist0ne/$imageName
  docker tag ist0ne/$imageName kubernetes/$imageName
done
```


## 通过kubeadm安装
```
kubeadm init --use-kubernetes-version v1.4.6
```
### 当通过kubeadm安装后，还需要安装网络
由于 pod 可能运行在不同的机器上，所以为了能让 pod 互相通信，就需要安装 pod 网络。这里使用的方案就是 weave net:
```
kubectl apply -f https://git.io/weave-kube
```
因为之前的 kube-dns addon 是依赖 pod 网络的，所以在没有部署 pod 网络之前，kube-dns 都会报错，因此只需要检查 kube-dns 是否成功就知道 pod 网络有没有成功了。
```
kubectl get pods --all-namespaces
```

## 如果docker hub也不能访问
如果docker hub也不能访问，那么可以通过daocloud的加速，它会在docker的配置--registry-mirro中加一个镜像服务器，但是通过它还是不能访问google container的镜像，所以还是需要上面在docker hub中配置
