---
layout: post
title: "Deploy Elasticsearch on bare-metal k8s(Part 2)"
date: 2021-02-24 17:00 +0900
tags: [kubernetes]
---


## 1. Introduction
* Elasticsearch 6.8을 Kubernetes에 설치
* Kubernetes는 Bare-Metal에 직접 설치한 버전
* 이번 글에서는 k8s에 Elasticsearch Cluster를 직접 구동
  * Elasticsearch가 사용할 Persistent Volume 설정
  * Elasticsearch를 구동할 Statefuleset 설정

## 2. Environment
* CentOS 7.9
* Kubernetes 1.18
* Elasticsearch 6.8
* MetalLB 0.9.5
* Calico 3.8.9

## 3. Persistent Volume Background
* Persistent Volume을 사용하기 전에 간략하게 알아본다
  * K8s의 파드 안 컨테이너에 있는 파일은 임시적이고 독립적
    * 장애가 발생해서 컨테이너가 사라지면, 내부 데이터가 손실
    * Pod 내 Container 간에 데이터 공유가 불가능
    * Volume을 도입하면, 이 문제를 해결
  * 볼륨은 기본적으로 파일시스템
    * 간단하게 디렉토리라고 생각하면 됨
    * K8s에서는 다양한 볼륨을 제공하고, 공식사이트에서 확인 가능
      * [Volume Types](https://kubernetes.io/ko/docs/concepts/storage/volumes/#volume-types)
    * 이번 글에서는 Local Volume을 사용
      * Elasticsearch는 Database이고, 이 성능을 극대화하려면 Local Volume을 사용하여 Spatial Locality을 극대화할 수 있음
      * [Spatial Locality(Locality of Reference)](https://en.wikipedia.org/wiki/Locality_of_reference)
  * 로컬 Persistent Volume을 Storage Class를 이용하여 정적 프로비저닝함
    * 동적 같은데, 정적이라고 표현하네요...?

## 4. Storage Class
* Volume을 사용하기 전 Storage Class를 구성(정의)
  * 관리자가 제공하는 스토리지의 "Class"(종류)를 설명함
  * 클래스를 통해 서비스 품질의 수준 또는 백업 정책 등 스토리지에 관한 프로파일 설정
    * 즉, 클래스에 속한 Volume의 프로파일이라고 할 수 있음
  * 스토리지클래스에는 provisioner, parameters, reclaimPolicy 필드가 포함되어 Volume의 행동을 묘사할 수 있음

* Storage Class 정의
  * provisioner는 [kubernetes.io/no-provisioner](http://kubernetes.io/no-provisioner)
  * reclaimPolicy는 Retain ([https://kubernetes.io/ko/docs/concepts/storage/storage-classes/#리클레임-정책](https://kubernetes.io/ko/docs/concepts/storage/storage-classes/#%EB%A6%AC%ED%81%B4%EB%A0%88%EC%9E%84-%EC%A0%95%EC%B1%85))
  * volumeBindingMode는 WaitForFirstConsumer ([https://kubernetes.io/ko/docs/concepts/storage/storage-classes/#볼륨-바인딩-모드](https://kubernetes.io/ko/docs/concepts/storage/storage-classes/#%EB%B3%BC%EB%A5%A8-%EB%B0%94%EC%9D%B8%EB%94%A9-%EB%AA%A8%EB%93%9C))
    * 굳이 immediate로 하여서, 미리 고정할 필요 없음

{% highlight yaml %}
# elastic-local-storage.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: elastic-local-storage
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
{% endhighlight %}

* Storage Class Apply & Describe
{% highlight shell %}
$ kubectl get sc
NAME                    PROVISIONER                    RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
elastic-local-storage   kubernetes.io/no-provisioner   Retain          WaitForFirstConsumer   false                  85m

$ kubectl describe sc elastic-local-storage
Name:            elastic-local-storage
IsDefaultClass:  No
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"elastic-local-storage"},"provisioner":"kubernetes.io/no-provisioner","reclaimPolicy":"Retain","volumeBindingMode":"WaitForFirstConsumer"}

Provisioner:           kubernetes.io/no-provisioner
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Retain
VolumeBindingMode:     WaitForFirstConsumer
Events:                <none>
{% endhighlight %}

## 5. Persistent Volume 구성
* spec.capacity.storage는 용량을 의미
* spec.accessModes는 pods와의 관계를 정의
  * [https://kubernetes.io/ko/docs/concepts/storage/persistent-volumes/#접근-모드](https://kubernetes.io/ko/docs/concepts/storage/persistent-volumes/#접근-모드)
  * ReadWriteOnce -- 하나의 노드에서 볼륨을 읽기-쓰기로 마운트할 수 있다
  * ReadOnlyMany -- 여러 노드에서 볼륨을 읽기 전용으로 마운트할 수 있다
  * ReadWriteMany -- 여러 노드에서 볼륨을 읽기-쓰기로 마운트할 수 있다
* spec.storageClassName: 위의 Storage Class Name과 동일하게 작성(elastic-local-storage)
  * 여러 노드를 사용할 경우(spec.nodeAffinity에 values가 여러 개인 경우), mount의 위치가 동일해야함
  * 또한 요청 용량은 mount 되어있는 Local Disk의 용량보다 커야한다
  * 현재 xfs, ext3, ext4일 때만 파일 크기 조정이 가능, [https://kubernetes.io/ko/docs/concepts/storage/persistent-volumes/#파일시스템을-포함하는-볼륨-크기-조정](https://kubernetes.io/ko/docs/concepts/storage/persistent-volumes/#파일시스템을-포함하는-볼륨-크기-조정)

{% highlight yaml %}
# elastic-local-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: elastic-local-pv1
spec:
  capacity:
    storage: 900Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: elastic-local-storage
  local:
    path: /data1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node3    # <- 이 부분은 각자 PC의 호스트네임을 사용하면 된다
{% endhighlight %}

* node3 PC에는 /data1에 932기가의 파티션이 mount되어 있다
{% highlight shell %}
$ df -Th
Filesystem     Type      Size  Used Avail Use% Mounted on
...
/dev/sdb1      xfs       932G   33M  932G   1% /data1
...
{% endhighlight %}

* Apply & Describe Persistent Volume 
{% highlight shell %}
$ kubectl apply -f persistent-volume.yaml
persistentvolume/elastic-local-pv created

$ kubectl get pv
kubectl get pv
NAME                CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                                    STORAGECLASS            REASON   AGE
elastic-local-pv1   900Gi      RWO            Retain           Available                                            elastic-local-storage            179m

$ kubectl describe pv elastic-local-pv1
Name:              elastic-local-pv1
Labels:            <none>
Annotations:       Finalizers:  [kubernetes.io/pv-protection]
StorageClass:      elastic-local-storage
Status:            Available
Claim:             
Reclaim Policy:    Retain
Access Modes:      RWO
VolumeMode:        Filesystem
Capacity:          900Gi
Node Affinity:     
  Required Terms:  
    Term 0:        kubernetes.io/hostname in [node3]
Message:           
Source:
    Type:  LocalVolume (a persistent volume backed by local storage on a node)
    Path:  /data1
Events:    <none>
{% endhighlight %}

## 6. Persistent Volume Claim 구성
* 이번 Elasticsearch에서는 volumeClaimTemplates을 이용하여 Persistent Volume Claim을 자동으로 생성
  * volumeClaimTemplates이 부분이 Persistent Volume Claim과 거의 동일하기 때문에 살펴본다
  * **일반적으로 데이터 베이스 Cluster를 구성할 때, PVC를 직접 사용하는 경우는 많지 않음**
* Persistent Volume Claim은 Pod이 Persistent Volume을 요청하기 위해 필요
  * 이 Claim을 통해 Pod에게 가장 적절한 Persistent Volume을 할당 받는다
  * storageClassName은 위 StroageClass 이름 사용
  * accessModes는 pv와 동일하게 설정
  * resources.requests.storage는 필요한 용량 서술
* StorageClass에서 volumeBindingMode가 WaitForFirstConsumer이기 때문에 Pending 상태
  * Pod이 PVC를 사용한다고 하면, Binding이 된다
{% highlight yaml %}
# elastic-local-pvc.yaml

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: elastic-local-pvc
spec:
  storageClassName: elastic-local-storage
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 900Gi
{% endhighlight %}

* Apply & Check PVC
{% highlight shell %}
$ kubectl apply -f persistent-volume-claim.yaml
persistentvolumeclaim/elastic-local-pvc created

$ kubectl get pvc
NAME                STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS            AGE
elastic-local-pvc   Pending                                      elastic-local-storage   6s
[root@control1 elastic]# kubectl describe pvc
Name:          elastic-local-pvc
Namespace:     default
StorageClass:  elastic-local-storage
Status:        Pending
Volume:
Labels:        <none>
Annotations:   <none>
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:
Access Modes:
VolumeMode:    Filesystem
Used By:       <none>
Events:
  Type    Reason                Age               From                         Message
  ----    ------                ----              ----                         -------
    Normal  WaitForFirstConsumer  7s (x2 over 21s)  persistentvolume-controller  waiting for first consumer to be created before binding
{% endhighlight %}

## 7. Statefulset
* 아직 Statefulset은 철학이나 설계를 확실하게 파악하지는 못 함..
  * Deployment와 같은 레벨로 보임
  * Statefulset을 실행하면, Replica의 수에 맞춰 Stateful한 Pod를 구동
  * 특히, 안정적이고 지속성을 갖는 애플리케이션에 적합

* 사용하기에 앞서 [제한사항](https://kubernetes.io/ko/docs/concepts/workloads/controllers/statefulset/)을 보는 것은 필수
  * 근데 봐도 아직은 잘 모르겠다.(2021/3/2)

* 아래 내용 중 중요한 부분은 env의 discovery.zen.ping.unicast.hosts, volumeClaimTemplate이다
  * spec.template.spec.env내 discovery.zen.ping.unicast.hosts에는 Part 1에서 만들었던 서비스를 사용
    * [Deploy Elasticsearch on bare-metal k8s(Part 1)](https://wonjaek36.github.io/2021/02/22/Deploy-Elasticsearch-on-bare-metal-k8s-part1.html) 내 elastic-cluster-svc
  * volumeClaimTemplates은 위의 PersistentVolumeClaim과 거의 흡사

{% highlight yaml %}
# statefulset.yaml 
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elastic-cluster

spec:
  serviceName: elasticsearch
  replicas: 3
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      terminationGracePeriodSeconds: 30
      containers:
      - name: elasticsearch
        image: docker.elastic.co/elasticsearch/elasticsearch:6.8.14
        ports:
        - containerPort: 9200
          name: rest
          protocol: TCP
        - containerPort: 9300
          name: inter-node
          protocol: TCP
        volumeMounts:
        - name: elastic-data
          mountPath: /usr/share/elasticsearch/data
        env:
          - name: cluster.name
            value: k8s-elastic
          - name: node.name
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: discovery.zen.ping.unicast.hosts
            value: elastic-cluster-svc
          - name: ES_JAVA_OPTS
            value: "-Xms4096m -Xmx4096m"
        #resources:
        #  limits:
        #    cpu
      initContainers:
      - name: fix-permissions
        image: busybox
        command: ["sh", "-c", "chown -R 1000:1000 /usr/share/elasticsearch/data"]
        securityContext:
          privileged: true
        volumeMounts:
        - name: elastic-data
          mountPath: /usr/share/elasticsearch/data
      - name: increase-vm-max-map
        image: busybox
        command: ["sysctl", "-w", "vm.max_map_count=262144"]
        securityContext:
          privileged: true
      - name: increase-fd-ulimit
        image: busybox
        command: ["sh", "-c", "ulimit -n 65536"]
        securityContext:
          privileged: true

  volumeClaimTemplates:
  - metadata:
      name: elastic-data
      labels:
        app: elasticsearch
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: elastic-local-storage
      resources:
        requests:
          storage: 900Gi
{% endhighlight %}

* Apply & Check
  * 실행 후 5일 지남
{% highlight shell %}
$ kubectl get sts
NAME              READY   AGE
elastic-cluster   3/3     5d21h

$ kubectl describe sts
Name:               elastic-cluster
Namespace:          default
CreationTimestamp:  Wed, 24 Feb 2021 15:58:14 +0900
Selector:           app=elasticsearch
Labels:             <none>
Annotations:        Replicas:  3 desired | 3 total
Update Strategy:    RollingUpdate
  Partition:        0
Pods Status:        3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=elasticsearch
  Init Containers:
   fix-permissions:
    Image:      busybox
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      chown -R 1000:1000 /usr/share/elasticsearch/data
    Environment:  <none>
    Mounts:
      /usr/share/elasticsearch/data from elastic-data (rw)
   increase-vm-max-map:
    Image:      busybox
    Port:       <none>
    Host Port:  <none>
    Command:
      sysctl
      -w
      vm.max_map_count=262144
    Environment:  <none>
    Mounts:       <none>
   increase-fd-ulimit:
    Image:      busybox
    Port:       <none>
    Host Port:  <none>
    Command:
      sh
      -c
      ulimit -n 65536
    Environment:  <none>
    Mounts:       <none>
  Containers:
   elasticsearch:
    Image:       docker.elastic.co/elasticsearch/elasticsearch:6.8.14
    Ports:       9200/TCP, 9300/TCP
    Host Ports:  0/TCP, 0/TCP
    Environment:
      cluster.name:                      k8s-elastic
      node.name:                          (v1:metadata.name)
      discovery.zen.ping.unicast.hosts:  elastic-cluster-svc
      ES_JAVA_OPTS:                      -Xms4096m -Xmx4096m
    Mounts:
      /usr/share/elasticsearch/data from elastic-data (rw)
  Volumes:  <none>
Volume Claims:
  Name:          elastic-data
  StorageClass:  elastic-local-storage
  Labels:        app=elasticsearch
  Annotations:   <none>
  Capacity:      900Gi
  Access Modes:  [ReadWriteOnce]
Events:          <none>

# Init Container는 Elasticsearch를 구성하기 전 환경설정을 하기 위한 부분
# Container에 포트가 9200/TCP, 9300/TCP가 열린 것을 확인

$ kubectl get po
NAME                READY   STATUS    RESTARTS   AGE
elastic-cluster-0   1/1     Running   0          5d22h
elastic-cluster-1   1/1     Running   0          5d21h
elastic-cluster-2   1/1     Running   0          5d21h

$ kubectl get pvc
NAME                             STATUS   VOLUME              CAPACITY   ACCESS MODES   STORAGECLASS            AGE
elastic-data-elastic-cluster-0   Bound    elastic-local-pv3   900Gi      RWO            elastic-local-storage   5d22h
elastic-data-elastic-cluster-1   Bound    elastic-local-pv1   900Gi      RWO            elastic-local-storage   5d21h
elastic-data-elastic-cluster-2   Bound    elastic-local-pv2   900Gi      RWO            elastic-local-storage   5d21h

$ kubectl get svc
NAME                       TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)          AGE
elastic-cluster-svc        ClusterIP      None          <none>         9300/TCP         5d22h
elastic-loadbalancer-svc   LoadBalancer   10.99.13.33   192.168.0.70   9200:32530/TCP   5d22h
kubernetes                 ClusterIP      10.96.0.1     <none>         443/TCP          6d13h

# Load-Balancer Service에 192.168.0.70으로 IP가 할당된 것을 확인

$ curl http://192.168.0.70:9200
{
  "name" : "elastic-cluster-0",
  "cluster_name" : "k8s-elastic",
  "cluster_uuid" : "DLfYiOwYTqOqYXFXdKKF2g",
  "version" : {
    "number" : "6.8.14",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "dab5822",
    "build_date" : "2021-02-02T19:58:04.182039Z",
    "build_snapshot" : false,
    "lucene_version" : "7.7.3",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}

# 한 번 더 수행해보면, 다른 Node에서 대답한 것을 확인할 수 있음
# 이는 Load-Balancer에 의해 Round-Robin 방식으로 요청이 들어가기 때문
$ curl http://192.168.0.70:9200
{
  "name" : "elastic-cluster-2",
  "cluster_name" : "k8s-elastic",
  "cluster_uuid" : "DLfYiOwYTqOqYXFXdKKF2g",
  "version" : {
    "number" : "6.8.14",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "dab5822",
    "build_date" : "2021-02-02T19:58:04.182039Z",
    "build_snapshot" : false,
    "lucene_version" : "7.7.3",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
{% endhighlight %}

## Reference
* Kubernetes 파드 공식문서: [https://kubernetes.io/ko/docs/concepts/workloads/pods/](https://kubernetes.io/ko/docs/concepts/workloads/pods/)
* Kubernetes 볼륨 공식문서: [https://kubernetes.io/ko/docs/concepts/storage/volumes/](https://kubernetes.io/ko/docs/concepts/storage/volumes/)
* Kubernetes 퍼시스턴트 공식문서: [https://kubernetes.io/ko/docs/concepts/storage/persistent-volumes/](https://kubernetes.io/ko/docs/concepts/storage/persistent-volumes/)
* Kubernetes Local Persistent Volume 예제: [https://kubernetes.io/ko/docs/tasks/configure-pod-container/configure-persistent-volume-storage/](https://kubernetes.io/ko/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)
* Kubernetes Storage Class: [https://kubernetes.io/ko/docs/concepts/storage/storage-classes/](https://kubernetes.io/ko/docs/concepts/storage/storage-classes/)
* Kubernetes Statefuleset: [https://kubernetes.io/ko/docs/concepts/workloads/controllers/statefulset/](https://kubernetes.io/ko/docs/concepts/workloads/controllers/statefulset/)
* Digital Ocean, How to Set up an Elasticsearch, Fluentd and Kibana(EFK) Logging Stack on Kubernetes: [https://www.digitalocean.com/community/tutorials/how-to-set-up-an-elasticsearch-fluentd-and-kibana-efk-logging-stack-on-kubernetes](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-elasticsearch-fluentd-and-kibana-efk-logging-stack-on-kubernetes)