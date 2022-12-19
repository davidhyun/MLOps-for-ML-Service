# Data Management

## DVC (Data Version Control)

- 데이터 버전 관리 도구 (오픈소스)
- 대부분의 스토리지와 호환 (Amazon S3, Google Drive, ...)
- GitHub 외에 GitLab, Bitbucket 등의 대부분의 Git 호스팅 서버와 연동
- Data Pipeline을 DAG로 관리
- Git과 유사한 인터페이스

![DVC](https://user-images.githubusercontent.com/64063767/208460661-1f2bdd0e-2e8c-4f3e-92dc-1c6acf3e9370.png)

## DVC Install (MacOS)

```bash
$ conda create -n dvc-tutorial python=3.9.6
$ conda activate dvc-tutorial
$ pip install dvc
# [all] option: remote storage로 s3, gs, azure, oss, ssh 모두 사용할 수 있도록 관련 패키지를 함께 설치
```

## DVC 저장소 설정

```bash
$ mkdir dvc-tutorial
$ git init
$ dvc init
```

## DVC 기본명령

### 1) data 생성

```bash
$ mkdir data
$ cd data
$ vi demo.txt
cat demo.txt
```

### 2) tracking

- `.dvc` 파일은 데이터의 메타정보를 가진 파일이다.
- git 에서는 파일 자체가 아닌, `.dvc` 파일만 관리하게 된다.

```bash
$ cd ..
$ dvc add data/demo.txt 	# demo.txt.dvc 파일 자동생성

# To track the changes with git
$ git add data/demo.txt.dvc data/.gitignore
```