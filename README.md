# ghost-in-k8s
Running ghost in Kubernetes(microk8s)

## Install microk8s

Refer to [MicroK8s - Fast, Light, Upstream Developer Kubernetes](https://microk8s.io/)

One command is enough to install microk8s

    $ sudo snap install microk8s --classic
    2019-05-18T09:43:53+09:00 INFO Waiting for restart...
    microk8s v1.14.1 from Canonical✓ installed

    $ sudo snap info microk8s
    name:      microk8s
    summary:   Kubernetes for workstations and appliances
    publisher: Canonical✓
    contact:   <https://github.com/ubuntu/microk8s>
    license:   unset
    description: |
      MicroK8s is a small, fast, secure, single node Kubernetes that installs on just about any Linux
      box. Use it for offline development, prototyping, testing, or use it on a VM as a small, cheap,
      reliable k8s for CI/CD. It's also a great k8s for appliances - develop your IoT apps for k8s and
      deploy them to MicroK8s on your boxes.
    commands:
      - microk8s.config
      - microk8s.ctr
      - microk8s.disable
      - microk8s.enable
      - microk8s.inspect
      - microk8s.istioctl
      - microk8s.kubectl
      - microk8s.reset
      - microk8s.start
      - microk8s.status
      - microk8s.stop
    services:
      microk8s.daemon-apiserver:          simple, enabled, active
      microk8s.daemon-apiserver-kicker:   simple, enabled, active
      microk8s.daemon-containerd:         simple, enabled, active
      microk8s.daemon-controller-manager: simple, enabled, active
      microk8s.daemon-etcd:               simple, enabled, active
      microk8s.daemon-kubelet:            simple, enabled, active
      microk8s.daemon-proxy:              simple, enabled, active
      microk8s.daemon-scheduler:          simple, enabled, active
    snap-id:      EaXqgt1lyCaxKaQCU349mlodBkDCXRcg
    tracking:     stable
    refresh-date: today at 09:44 KST
    channels:
      stable:         v1.14.1         2019-04-18 (522) 214MB classic
      candidate:      v1.14.1         2019-04-15 (522) 214MB classic
      beta:           v1.14.1         2019-04-15 (522) 214MB classic
      edge:           v1.14.2         2019-05-17 (604) 217MB classic
      1.15/stable:    –
      1.15/candidate: –
      1.15/beta:      –
      1.15/edge:      v1.15.0-alpha.3 2019-05-08 (578) 215MB classic
      1.14/stable:    v1.14.1         2019-04-18 (521) 214MB classic
      1.14/candidate: v1.14.1         2019-04-15 (521) 214MB classic
      1.14/beta:      v1.14.1         2019-04-15 (521) 214MB classic
      1.14/edge:      v1.14.2         2019-05-17 (603) 217MB classic
      1.13/stable:    v1.13.5         2019-04-22 (526) 237MB classic
      1.13/candidate: v1.13.6         2019-05-09 (581) 237MB classic
      1.13/beta:      v1.13.6         2019-05-09 (581) 237MB classic
      1.13/edge:      v1.13.6         2019-05-08 (581) 237MB classic
      1.12/stable:    v1.12.8         2019-05-02 (547) 259MB classic
      1.12/candidate: v1.12.8         2019-05-01 (547) 259MB classic
      1.12/beta:      v1.12.8         2019-05-01 (547) 259MB classic
      1.12/edge:      v1.12.8         2019-04-24 (547) 259MB classic
      1.11/stable:    v1.11.10        2019-05-10 (557) 258MB classic
      1.11/candidate: v1.11.10        2019-05-02 (557) 258MB classic
      1.11/beta:      v1.11.10        2019-05-02 (557) 258MB classic
      1.11/edge:      v1.11.10        2019-05-01 (557) 258MB classic
      1.10/stable:    v1.10.13        2019-04-22 (546) 222MB classic
      1.10/candidate: v1.10.13        2019-04-22 (546) 222MB classic
      1.10/beta:      v1.10.13        2019-04-22 (546) 222MB classic
      1.10/edge:      v1.10.13        2019-04-22 (546) 222MB classic
    installed:        v1.14.1                    (522) 214MB classic

## Enable services(microk8s)

    $ sudo microk8s.enable dashboard registry dns
    Enabling dashboard
    secret/kubernetes-dashboard-certs created
    serviceaccount/kubernetes-dashboard created
    deployment.apps/kubernetes-dashboard created
    service/kubernetes-dashboard created
    service/monitoring-grafana created
    service/monitoring-influxdb created
    service/heapster created
    deployment.extensions/monitoring-influxdb-grafana-v4 created
    serviceaccount/heapster created
    configmap/heapster-config created
    configmap/eventer-config created
    deployment.extensions/heapster-v1.5.2 created
    dashboard enabled
    Enabling the private registry
    Enabling default storage class
    deployment.extensions/hostpath-provisioner created
    storageclass.storage.k8s.io/microk8s-hostpath created
    Storage will be available soon
    Applying registry manifest
    namespace/container-registry created
    persistentvolumeclaim/registry-claim created
    deployment.extensions/registry created
    service/registry created
    The registry is enabled
    Enabling DNS
    Applying manifest
    service/kube-dns created
    serviceaccount/kube-dns created
    configmap/kube-dns created
    deployment.extensions/kube-dns created
    Restarting kubelet
    DNS is enabled

서비스 상태는 `microk8s.status`로 확인 가능 

    $ sudo microk8s.status
    [sudo] password for cychong:
    microk8s is running
    addons:
    jaeger: disabled
    fluentd: disabled
    gpu: disabled
    storage: enabled
    registry: enabled
    ingress: disabled
    dns: enabled
    metrics-server: disabled
    prometheus: disabled
    istio: disabled
    dashboard: enabled


## Setup volume

ghost의 DB를 sqlite3를 사용하고 있어 DB 파일이 저장될 위치인 공간을 node에 생성한다. 

### volume.yaml

    kind: PersistentVolume
    apiVersion: v1
    metadata:
      name: ghost-pv-volume
      labels:
        type: local
    spec:
      storageClassName: manual
      capacity:
        storage: 12Gi
      accessModes:
        - ReadWriteOnce
      hostPath:
        path: "/home/cychong/Dropbox/Apps/ghost/content"

    /Dropbox/working/kubernetes$ sudo microk8s.kubectl apply -f volume.yaml
    [sudo] password for cychong:
    persistentvolume/ghost-volume created
    
    /Dropbox/working/kubernetes$ sudo microk8s.kubectl get pv
    NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                               STORAGECLASS        REASON   AGE
    ghost-volume                               12Gi       RWO            Retain           Available                                       manual                       17s
    pvc-942cf467-7906-11e9-8f0b-00264a162bca   20Gi       RWX            Delete           Bound       container-registry/registry-claim   microk8s-hostpath            28h
    
    
특정 volume 정보만 확인하려면 `get pv` 뒤에 volume 이름을 지정한다.

    /Dropbox/working/kubernetes$ sudo microk8s.kubectl get pv ghost-volume
    NAME           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
    ghost-volume   12Gi       RWO            Retain           Available           manual                  24s

특정 volume 삭제하려면 `delete pv [volume-name]`을 사용한다. 

    /Dropbox/working/kubernetes$ sudo microk8s.kubectl get pv
    NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                               STORAGECLASS        REASON   AGE
    ghost-pv-volume                            12Gi       RWO            Retain           Available                                       manual                       4s
    ghost-volume                               12Gi       RWO            Retain           Available                                       manual                       20m
    pvc-942cf467-7906-11e9-8f0b-00264a162bca   20Gi       RWX            Delete           Bound       container-registry/registry-claim   microk8s-hostpath            28h
    
    /Dropbox/working/kubernetes$ sudo microk8s.kubectl delete pv ghost-volume
    persistentvolume "ghost-volume" deleted
    
    /Dropbox/working/kubernetes$ sudo microk8s.kubectl get pv
    NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                               STORAGECLASS        REASON   AGE
    ghost-pv-volume                            12Gi       RWO            Retain           Available                                       manual                       112s
    pvc-942cf467-7906-11e9-8f0b-00264a162bca   20Gi       RWX            Delete           Bound       container-registry/registry-claim   microk8s-hostpath            28h

## Volume Claim

volume-claim.yaml

Pod에서 PV를 사용하려면 PVC(Persistent Volume Claim)을 설정한다. 앞에서 본 `volume.yaml`과 유사하지만 `requests`를 통해 일정 크기(아래는 10G)를 요청한다. 

    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: ghost-pv-claim
      labels:
        type: local
    spec:
      storageClassName: manual
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 10Gi

    /Dropbox/working/kubernetes$ sudo microk8s.kubectl apply -f volume-claim.yaml
    persistentvolumeclaim/ghost-pv-claim created

### Claim 된 Volume의 상태 확인

 `get pv` 명령으로 volume의 상태를 확인하면 이전에는 `STATUS`가 `Available`이었던 것이 `Bound`로 변경된 것을 알 수 있다.

    /Dropbox/working/kubernetes$ sudo microk8s.kubectl get pv ghost-pv-volume
    NAME              CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   REASON   AGE
    ghost-pv-volume   12Gi       RWO            Retain           Bound    default/ghost-pv-claim   manual                  3m40s

 `get pvc` 명령으로 확인하면 claim된 volume에 대한 정보를 확인할 수 있다.

    $ sudo microk8s.kubectl get pvc
    NAME             STATUS   VOLUME            CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    ghost-pv-claim   Bound    ghost-pv-volume   12Gi       RWO            manual         146m

## Deploy

이제 실제 container를 이용한 Pod를 구성한다.

Pod를 직접 생성하고 Container를 실행할 수도 있지만, 이 경우 Pod의 상태를 확인하여 다시 실행해 주는 kubernetes의 관리 기능을 이용할 수 없다. 

### deployment.yaml

동시에 하나의 ghost container만 띄우면 되므로 아래와 같이 `replicas`를 1로 지정한다. 

그 외 deploy할 때 항상 최신 docker image를 다운 받도록 `imagePullPolicy`를 `Always` 로 설정한다.

Container에 넘길 환경 변수 등은 `env` 항목을 통해 넘길 수 있고, 위에서 생성한 volume cliam을 ghost-content라는 이름으로 지정한 후 `mountPath`를 이용하여 container의 특정 위치에 마운트되도록 한다.

즉 `ghost-pv-volume` → `ghost-pv-claim` → `ghost-content`→ `ghost`

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: ghost
      labels:
        app: ghost
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: ghost
      template:
        metadata:
          labels:
            app: ghost
        spec:
          containers:
          - name: ghost
            image: ghost
            imagePullPolicy: Always
            ports:
            - containerPort: 2368
            env:
            - name: url
              value: http://sosa0sa.com:2368
            - name: NODE_ENV
              value: production
            volumeMounts:
            - mountPath: /var/lib/ghost/content
              name: ghost-content
          volumes:
          - name: ghost-content
            persistentVolumeClaim:
              claimName: ghost-pv-claim

    /Dropbox/working/kubernetes$ sudo microk8s.kubectl apply -f deployment.yaml
    [sudo] password for cychong:
    deployment.apps/ghost created
    
    /Dropbox/working/kubernetes$ sudo microk8s.kubectl get pods
    NAME                     READY   STATUS              RESTARTS   AGE
    ghost-79b8c8979d-7m7fm   0/1     ContainerCreating   0          9s
    
    /Dropbox/working/kubernetes$ sudo microk8s.kubectl get deploy
    NAME    READY   UP-TO-DATE   AVAILABLE   AGE
    ghost   0/1     1            0           71s

시간이 지나면 READY 상태가 `1/1`로 변경되고, `AVAILABLE` 값이 역시 0에서 1로 변경된다. 

    /Dropbox/working/kubernetes$ sudo microk8s.kubectl get pods
    NAME                     READY   STATUS    RESTARTS   AGE
    ghost-79b8c8979d-7m7fm   1/1     Running   0          90s
    /Dropbox/working/kubernetes$ sudo microk8s.kubectl get deploy
    NAME    READY   UP-TO-DATE   AVAILABLE   AGE
    ghost   1/1     1            1           92s

이제 Pod/container는 정상적으로 실행이 된 상태

만일  `ghost`라고 명명된 Pod이 비정상 상태가 되면 자동으로 다른 Pod를 실행시킨다. 

    /work/ghost-in-k8s$ sudo microk8s.kubectl get pods
    NAME                                    READY   STATUS        RESTARTS   AGE
    default-http-backend-5769f6bc66-jpcfj   1/1     Terminating   0          27h
    ghost-79b8c8979d-7m7fm                  1/1     Running       0          29h
    
    /work/ghost-in-k8s$ sudo microk8s.kubectl delete pod ghost-79b8c8979d-7m7fm
    pod "ghost-79b8c8979d-7m7fm" deleted
    
    ^Z
    [1]+  Stopped                 sudo microk8s.kubectl delete pod ghost-79b8c8979d-7m7fm
    /work/ghost-in-k8s$ bg
    [1]+ sudo microk8s.kubectl delete pod ghost-79b8c8979d-7m7fm &
    
    /work/ghost-in-k8s$ sudo microk8s.kubectl get pod
    NAME                                    READY   STATUS        RESTARTS   AGE
    default-http-backend-5769f6bc66-jpcfj   1/1     Terminating   0          27h
    ghost-79b8c8979d-468d2                  1/1     Running       0          76s
    ghost-79b8c8979d-7m7fm                  1/1     Terminating   0          29h

위 예에서는 기존에 동작하고 있던 pod을 삭제하니(정상적으로 삭제가 되지 않아 `STATUS`가 `Terminating` 상태로 표시되고 있다. 확인 필요)

`kind: Pod` or `kind: deployment`?

Pod를 직접 생성하는 yaml 파일을 만들어 사용할 수도 있지만 상용에서는 직접 pod를 만들기 보다 deployment를 통해 pod를 생성하고 pod를 관리한다. 

[In kubernetes what is the difference between a pod and a deployment?](https://stackoverflow.com/questions/41325087/in-kubernetes-what-is-the-difference-between-a-pod-and-a-deployment)

## Service

ghost는 Cluster 외부에서 접속이 필요한 블로그 서비스이므로 외부에서 접속이 가능하도록 external IP를 가질 수 있게 service를 구성한다. 

### service.yaml

    apiVersion: v1
    kind: Service
    metadata:
      name: ghost
    spec:
      type: LoadBalancer
      selector:
        app: ghost
      ports:
      - protocol: TCP
        port: 2368
        targetPort: 2368

`type:LoadBalancer`를 적용하면 외부 통신에 대해 Load Balancer를 사용하여 자동으로 EXTERNAL-IP가 설정된다고 한다.

[https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)

> LoadBalancer: Exposes the service externally using a cloud provider’s load balancer. NodePort and ClusterIP services, to which the external load balancer will route, are automatically created.

[How to install Kubernetes dashboard on external IP address?](https://stackoverflow.com/questions/49869989/how-to-install-kubernetes-dashboard-on-external-ip-address)

하지만 위 파일을 적용했는데 `EXTERNAL-IP` 정보가 pending으로 나온다. 

    /Dropbox/working/kubernetes$ sudo microk8s.kubectl apply -f service.yaml
    service/ghost created
    /Dropbox/working/kubernetes$ sudo microk8s.kubectl get service
    NAME         TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
    ghost        LoadBalancer   10.152.183.119   <pending>     2368:30126/TCP   3s
    kubernetes   ClusterIP      10.152.183.1     <none>        443/TCP          32h

웹 브라우저를 이용해서 Cluster 외부에서 접속(ghost Pod가 deploy된 mini1 머신의 IP로 접속)을 시도해 보지만 접속이 안된다.

아무래도 저 `pending` 상태로 나오는 게 이상한데 혹시나 하고 검색을 해 보니 역시나 저렇게 되면 외부에서 접속이 안된다고. 그 이유로 처음에 사용한 service 파일에서 사용된 `LoadBalancer` type이 microk8s에서는 지원되지 않기 때문이라고 한다. 추가로 load balancer를 설치해도 여전히 안된다고...

> ktsakalozos commented on Nov 23, 2018
Hi @khteh ,
Kubernetes does not provide a loadbalancer. It is assumed that loadbalancers are an external component [1]. MicroK8s is not shipping any loadbalancer but even if it did there would not have been any nodes to balance load over. There is only one node so if you want to expose a service you should use the NodePort service type.

### microk8s does not support `LoadBalancer`

[nginx loadbalancer service EXTERNAL-IP is always in "pending" state · Issue #200 · ubuntu/microk8s](https://github.com/ubuntu/microk8s/issues/200)

[kubernetes service external ip pending](https://stackoverflow.com/questions/44110876/kubernetes-service-external-ip-pending)

### Publishing Service Type 변경

service.yaml 파일의 Publishing Service Type을 microk8s에서 지원되는 `NodePort`로 변경한 후 다시 appy한 후 다시 접속을 시도하지만 여전히 접속이 되지 않는다.

### Service 수정해서 External IP 강제 지정

Node의 `EXTERNAL-IP` 정보가 여전히 빈칸으로 나온다. 

    /Dropbox/working/kubernetes$ sudo microk8s.kubectl get nodes -o wide
    NAME    STATUS   ROLES    AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
    mini1   Ready    <none>   35h   v1.14.1   192.168.1.100   <none>        Ubuntu 18.04.2 LTS   4.15.0-48-generic   containerd://1.2.5

원래(?) service 에서 NodePort 타입으로 지정하고 port 만 지정하면 Cluster 외부에서 접속이 되어야 하는데 이것 때문에 안되는 듯 하다. 

그래서 결국 임시로 service.yaml 수정해서 external IP를 강제로 지정했다. `exteranlIPs`이므로 배열 형태로 지정 한다. 

    apiVersion: v1
    kind: Service
    metadata:
      name: ghost
    spec:
      type: NodePort
      externalTrafficPolicy : Local
      externalIPs : [192.168.1.100]
      selector:
        app: ghost
      ports:
      - protocol: TCP
        port: 2368
        targetPort: 2368

변경된 service.yaml을 적용한 후 service 내용을 확인하면 `EXTERNAL-IP` 정보가 변경된 것을 확인할 수 있다. 

    /work/ghost-in-k8s$ sudo microk8s.kubectl get svc -o wide
    NAME         TYPE        CLUSTER-IP       EXTERNAL-IP     PORT(S)          AGE     SELECTOR
    ghost        NodePort    10.152.183.119   192.168.1.100   2368:30126/TCP   4h54m   app=ghost
    kubernetes   ClusterIP   10.152.183.1     <none>          443/TCP          37h     <none>

이제 Cluster 외부에서도 node의 EXTERNAL-IP를 이용해서 접속할 수 있다. 


# Reference

- [How to run ghost in Kubernetes](https://mendoza.io/how-to-run-ghost-in-kubernetes/amp/) - use the YAML files in this post for my setup  
- [Exposing an External IP Address to Access an Application in a Cluster](https://kubernetes.io/docs/tutorials/stateless-application/expose-external-ip-address/)
- [Configure a Pod to Use a PersistentVolume for Storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)
