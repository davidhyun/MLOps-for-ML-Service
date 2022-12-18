# MacOS M1 Kubernetes 환경구축 (2022.5 기준)

## 1. Homebrew install

```bash
$ brew install minikube
```

- 맥의 강점이다. Homebrew를 통해 어떤 패키지든 간편 설치가 가능하다. 하지만 자동적으로 최신버전을 설치해준다.

## 2. Curl install

- 직접 binary 다운로드로 minikube 설치 (버전선택 가능)

- MacOS M1에서 버전 이슈가 있었다. **minikube 1.25.1 버전을 설치**해야 한다.
- 반드시 ~($HOME) 경로를 기준으로 설치를 진행해야 한다.

```bash
$ cd ~
$ curl -Lo minikube https://github.com/kubernetes/minikube/releases/download/v1.25.1/minikube-darwin-arm64 \
  && chmod +x minikube
$ sudo install minikube /usr/local/bin/minikube
$ minikube version # v1.25.1
```

## 3. minikube start

- minikube를 실행할 가상 드라이버로 Hyperkit, Hypoer-V, Docker, VirtualBox 중 M1이 사용할 수 있는 드라이버는 Docker 드라이버가 유일하다.

```bash
$ minikube start --driver=docker
```

## 4. kubectl install

- kubernetes control 약자로서, 쿠버네티스 커맨드 CLI 도구이다. kubectl을 통해 kubernetes 클러스터에 명령어를 전달할 수 있다.

```bash
$ brew install kubectl
```

<br>

## References

- [macOs m1 환경에서 Kubernetes 시작하기(feat. Docker)](https://velog.io/@pinion7/macOs-m1-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C-kubernetes-%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0)