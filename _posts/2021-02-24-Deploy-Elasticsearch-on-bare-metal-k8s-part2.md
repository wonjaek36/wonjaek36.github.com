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
* volumeClaimTemplates이 부분이 Persistent Volume Claim과 거의 동일


## Reference
* Kubernetes 파드 공식문서: [https://kubernetes.io/ko/docs/concepts/workloads/pods/](https://kubernetes.io/ko/docs/concepts/workloads/pods/)
* Kubernetes 볼륨 공식문서: [https://kubernetes.io/ko/docs/concepts/storage/volumes/](https://kubernetes.io/ko/docs/concepts/storage/volumes/)
* Kubernetes 퍼시스턴트 공식문서: [https://kubernetes.io/ko/docs/concepts/storage/persistent-volumes/](https://kubernetes.io/ko/docs/concepts/storage/persistent-volumes/)
* Kubernetes Local Persistent Volume 예제: [https://kubernetes.io/ko/docs/tasks/configure-pod-container/configure-persistent-volume-storage/](https://kubernetes.io/ko/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)
* Kubernetes Storage Class: [https://kubernetes.io/ko/docs/concepts/storage/storage-classes/](https://kubernetes.io/ko/docs/concepts/storage/storage-classes/)