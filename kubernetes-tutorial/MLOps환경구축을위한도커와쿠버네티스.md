# MLOps 환경구축을 위한 도커와 쿠버네티스

## 0. MLOps 필요 조건

- Reproducibility : 실행 환경의 일관성 & 독립성
- Job Scheduling : 스케줄 관리, 병렬 작업 관리, 유휴 자원 관리
- Auto-healing & Auto-scaling : 장애 대응, 트래픽 대응

<br/>

## 1. Docker

> Build Once, Run Anywhere

- Containerization : 컨테이너화 하는 기술
- Container : 격리된 공간에서 **프로세스**를 실행시킬 수 있는 기술

## 2. Kubernetes

> Container Orchestration

- Desired State : 원하는 **상태**를 기술하는 선언형 인터페이스 (Kubernetes Native)
- Master & Worker Node

![image](https://user-images.githubusercontent.com/64063767/208122315-bbed4d3f-0c56-487c-aac8-d990a92ad969.png)

### 2.0 Kubernetes Resource

- Pod, Deployment, Service, PVC ...

### 2.1 Pod

- Pod는 쿠버네티스에서 생성하고 관리할 수 있는 배포 가능한 **가장 작은 컴퓨팅 단위**이다.
- 쿠버네티스는 Pod 단위로 스케줄링, 로드밸런싱, 스케일링 등의 관리 작업을 수행한다.
  - 쿠버네티스에 어떤 어플리케이션을 배포하고 싶다면 최소 Pod 단위로 구성해야 한다는 의미이다.
- Pod는 Container를 감싼 개념으로 볼 수 있다.
  - 하나의 Pod는 한개의 Container 또는 여러 개의 Container 로 구성될 수 있다.
  - Pod 내부의 여러 Container는 자원을 공유한다.
- Pod는 Stateless한 특징이 있으며, **언제든지 삭제될 수 있는 자원**이다.

#### 2.1.1 Pod 생성

```yaml
# <pod.yaml>
# kubernetes resource's API Version
apiVersion: v1

# kubernetes resource name
kind: Pod

# metadata: name, namespace, labels, annotations 등을 포함
metadata:
  name: counter
spec:
  containers:
    - name: count				# 컨테이너 이름
      image: busybox 		# 컨테이너 이미지
      # entrypoint의 arguments로 입력하고 싶은 부분
      args: [/bin/sh, -c, 'i=0; while true; do echo "$i: $(data)"; i=$((i+1)); sleep 1; done']
```

```bash
$ kubectl apply -f <pod.yaml>
```

#### 2.1.2 Pod 조회

```bash
$ kubectl get pod																		# 전체 pod 조회
$ kubectl get pod <pod-name>												# 특정 pod 조회
$ kubectl describe pod <pod-name>										# 특정 pod 상세조회
$ kubectl get pod -o wide														# pod 목록 상세조회
$ kubectl get pod <pod-name> -o yaml								# pod를 yaml 형식으로 출력
$ kubectl get pod -w																# kubectl get pod의 결과를 계속 보여주며, 변화가 있을 때만 업데이트
$ kubectl logs -f <pod-name>												# pod 로그 실시간 조회
$ kubectl logs -f <pod-name> -c <container-name>		# pod의 특정 컨테이너 로그 조회
```

- 조회 결과는 Desired State가 아닌 **Current State**
- namespace?
  - namespace는 kubernetes에서 리소스를 격리하는 가상의(논리적인) 단위
  - `kubectl config view --minify | grep namespace:` 명령어로 current namespace가 어떤 namespace로 설정되었는지 확인할 수 있다. 따로 설정하지 않았다면 `default` 로 namespace가 기본으로 설정되어 있을 것이다.
- 특정 namespace 또는 모든 namespace의 pod를 조회할 수도 있다.

```bash
# kube-system namespace의 pod를 조회
$ kubectl get pod -n kube-system

# 모든 namespace의 pod를 조회
$ kubectl get pod -A
```

#### 2.1.3 Pod 내부접속

```bash
$ kubectl exec -it <pod-name> -- sh
```

#### 2.1.4 Pod 삭제

```bash
# pod 삭제
$ kubectl delete <pod-name>

# 리소스를 생성할 때 사용된 yaml 파일을 사용해서 리소스 삭제
$ kubectl delete -f <pod.yaml>
```

<br>

### 2.2 Deployment

- Deployment는 Pod와 Replicaset에 대한 **관리**를 제공하는 단위이다.
- 관리라는 의미에는 Self-healing, Scaling, Rollout(무중단 업데이트)과 같은 기능을 포함한다.
- Deploymentsms Pod를 감싼 개념으로 볼 수 있다.
  - Pod를 Deployment로 배포함으로써 여러 개로 복제된 Pod, 여러 버전의 Pod를 안전하게 관리할 수 있다.

#### 2.2.1 Deployment 생성

```yaml
# <deployment.yaml>
# kubernetes resource's API Version
apiVersion: apps/v1

# kubernetes resource name
kind: Deployment

# metadata: name, namespace, labels, annotations 등을 포함
metadata:
  name: nginx-deployment
  labels:
    app: nginx

# 메인파트: resource의 desired state를 명시
spec:
  replicas: 3 # 동일한 템플릿의 pod를 3개 복제본으로 생성
  selector:
    matchLabels:
      app: nginx
  template: # Pod의 템플릿
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
          - containerPort: 80 # 컨테이너 내부 포트
```

```bash
$ kubectl apply -f <deployment.yaml>
```

#### 2.2.2 Deployment 조회

```bash
$ kubectl get deployment
$ kubectl get deployment,pod
```

#### 2.2.3 Deployment Auto-healing

```bash
# pod 삭제
$ kubectl delete pod <pod-name>

# 기존 pod가 삭제되고, 동일한 pod가 새로 생성됨
$ kubectl get pod
```

#### 2.2.4 Deployment Scaling

```bash
# replica 개수 변경
$ kubectl scale deployment/nginx-deployment --replicas=5
```

#### 2.2.5 Deployment 삭제

```bash
$ kubectl delete deployment <deployment-name>
$ kubectl delete -f <deployment.yaml>
```

<br>

### 2.3 Service

- Service는 쿠버네티스에 배포한 **어플리케이션(Pod)을 외부에서 접근하기 쉽게 추상화한 리소스**이다.
- Pod는 IP를 할당받고 생성되지만, 언제든지 죽었다가 다시 살아날 수 있으며, 그 과정에서 IP를 항상 재할당 받기때문에 고정된 IP로 원하는 Pod에 접근할 수 없다.
- 따라서 클러스터 외부 혹은 내부에서 Pod에 접근할 때는 Pod의 IP가 아닌 Service를 통해서 접근하는 방식을 거친다.
- Service는 고정된 IP를 가지며, Service는 하나 혹은 여러 개의 Pod와 매칭된다.
- 따라서 클라이언트가 Service의 주소로 접근하면, 실제로는 Service에 매칭된 Pod에 접속할 수 있게 된다.

#### 2.3.1 Service 생성

- 할당된 \<Pod-IP>는 클러스터 내부에서만 접근할 수 있는 IP이기 때문에 외부에서는 Pod에 접속할 수 없다.

```bash
# 생성된 Pod의 IP 조회
$ kubectl get pod -o wide

# 외부에서 통신 테스트
$ curl -X GET <Pod-IP> -vvv
$ ping <Pod-IP>
```

- minikube 내부에서 통신 테스트

```bash
# minikube 내부에서 통신 테스트
$ minikube ssh

$ curl -X GET <Pod-IP> -vvv
$ ping <Pod-IP>
```

```yaml
# <service.yaml>
apiVersion: v1
kind: Service
metadata:
  name: my-nginx
  labels:
    run: my-nginx
spec:
  type: NodePort # Service Type
  ports:
  - port: 80
    protocol: TCP
  selector: # 아래 label을 가진 Pod를 매핑
    app: nginx
```

```bash
$ kubectl apply -f <service.yaml>
```

#### 2.3.2 Service 조회

```bash
$ kubectl get service 	# PORT 80:<PORT> 숫자 확인

# minikube 자체를 노드가 하나인 클러스터라고 본다면
$ minikube ip # 192.168.49.2
$ curl -X GET $(minikube ip):<PORT>		# 서비스를 통해서 클러스터 외부에서도 정상적으로 pod에 접근 가능
```

- Service Type?
  - `NodePort`라는 type을 사용했기 때문에, 클러스터 외부에서 minikube라는 kubernetes cluster 내부에 배포된 서비스에 접근할 수 있었다.
    - 접근하는 IP는 Pod가 생성된 노드(머신)의 IP를 사용하고, Port는 할당받은 Port를 사용한다.
  - `LoadBalancer`라는 type을 사용해도, 마찬가지로 클러스터 외부에서 접근할 수 있지만, Loadbalancer를 사용하기 위해서는 LoadBalancing 역할을 하는 모듈이 추가적으로 필요하다.
  - `ClusterIP`라는 type은 고정된 IP, PORT를 제공하지만, 클러스터 내부에서만 접근할 수 있는 대역의 주소가 할당된다.
  - 실무에서는 주로 kubernets cluster에 MetalLB와 같은 LoadBalancing 역할을 하는 모듈을 설치한 후, LoadBalancer type으로 서비스를 expose 하는 방식을 사용한다. 

<br>

### 2.4 PVC

- Persistent Volume(PV), Persistent Volume Claim(PVC)은 stateless한 Pod가 영구적으로(persistent) **데이터를 보존**하고 싶은 경우 사용하는 리소스이다.
- Docker의 볼륨과 유사한 역할을 한다.
- PV는 관리자가 생성한 실제 저장 공간의 정보를 담고 있고, PVC는 사용자가 요청한 저장 공간의 스펙에 대한 정보를 담고 있는 리소스이다.
  - Pod 내부에 작성한 데이터는 기본적으로 언제든지 사라질 수 있기에, 보존하고 싶은 데이터가 있다면 Pod에 PVC를 mount해서 사용해야 한다.
- PVC를 사용하면 여러 Pod 간의 data 공유도 쉽게 가능하다.

#### 2.4.1 PVC 생성

- minikube를 생성하면, 기본적으로 minikube와 함께 설치되는 storageclass가 존재한다. `kubectl get storageclass`
- PVC를 생성하면 해당 PVC의 스펙에 맞는 PV를 그 즉시 자동으로 생성해준 뒤, PVC와 매칭시켜준다. (dynamic provisioning 지원하는 storageclass)

```yaml
# <pvc.yaml>
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    # ReadWriteOnce, ReadWriteMany
    - ReadWriteMany
  volumeMode: Filesystem
  resources:
    requests:
      storage: 10Mi # storage 용량 설정
  storageClassName: standard
```

```bash
$ kubectl apply -f <pvc.yaml>
```

#### 2.4.2 PVC 조회

```bash
$ kubectl get pvc,pv
```

#### 2.4.3 Pod에서 PVC 사용

```yaml
# <pod-pvc.yaml>
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd	# 어떤 이름이든 상관없으나, 아래 volumes[0].name과 일치해야 함
  volumes:
    - name: mypd		# 어떤 이름이든 상관없으나, 위의 volumeMounts[0].name과 일치해야 함
      persistentVolumeClaim:
        claimName: myclaim	# mount할 pvc 이름
```

```bash
# Pod 삭제 및 재생성시 데이터 보존 여부 확인
$ kubectl apply -f pod-pvc.yaml
$ kubectl exec -it mypod -- bash

$ touch /hello.txt
$ touch /var/www/html/hello.txt
$ exit

$ kubectl delete pod mypod
$ kubectl apply -f pod-pvc.yaml
$ ls -la /var/www/html
```