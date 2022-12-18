# MLOps 환경구축을 위한 도커와 쿠버네티스

## 0. MLOps 필요 조건

- Reproducibility : 실행 환경의 일관성 & 독립성
- Job Scheduling : 스케줄 관리, 병렬 작업 관리, 유휴 자원 관리
- Auto-healing & Auto-scaling : 장애 대응, 트래픽 대응

<br/>

## Docker

> Build Once, Run Anywhere

- Containerization : 컨테이너화 하는 기술
- Container : 격리된 공간에서 **프로세스**를 실행시킬 수 있는 기술

## Kubernetes

> Container Orchestration

- Desired State : 원하는 **상태**를 기술하는 선언형 인터페이스 (Kubernetes Native)
- Master & Worker Node

![image](https://user-images.githubusercontent.com/64063767/208122315-bbed4d3f-0c56-487c-aac8-d990a92ad969.png)

### Pod 란?

- Pod는 쿠버네티스에서 생성하고 관리할 수 있는 배포 가능한 **가장 작은 컴퓨팅 단위**이다.
- 쿠버네티스는 Pod 단위로 스케줄링, 로드밸런싱, 스케일링 등의 관리 작업을 수행한다.
  - 쿠버네티스에 어떤 어플리케이션을 배포하고 싶다면 최소 Pod 단위로 구성해야 한다는 의미이다.
- Pod는 Container를 감싼 개념
  - 하나의 Pod는 한개의 Container 또는 여러 개의 Container 로 구성될 수 있다.
  - Pod 내부의 여러 Container는 자원을 공유한다.
- Pod는 Stateless한 특징이 있으며, **언제든지 삭제될 수 있는 자원**이다.

### Pod 생성

```yaml
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
$ kubectl apply -f <yaml-file-path>
```

### Pod 조회

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

### Pod 내부접속

```bash
$ kubectl exec -it <pod-name> -- sh
```

### Pod 삭제

```bash
# pod 삭제
$ kubectl delete <pod-name>

# 리소스를 생성할 때 사용된 yaml 파일을 사용해서 리소스 삭제
$ kubectl delete -f <yaml-file-path>
```