# Data Management

## 1. DVC (Data Version Control)

- 데이터 버전 관리 도구 (오픈소스)
- 대부분의 스토리지와 호환 (Amazon S3, Google Drive, ...)
- GitHub 외에 GitLab, Bitbucket 등의 대부분의 Git 호스팅 서버와 연동
- Data Pipeline을 DAG로 관리
- Git과 유사한 인터페이스

![DVC](https://user-images.githubusercontent.com/64063767/208460661-1f2bdd0e-2e8c-4f3e-92dc-1c6acf3e9370.png)

<br/>

## 2. DVC Install (MacOS)

```bash
$ conda create -n dvc-tutorial python=3.9.6
$ conda activate dvc-tutorial
$ pip install 'dvc[all]'
# [all] option: remote storage로 s3, gs, azure, oss, ssh 모두 사용할 수 있도록 관련 패키지를 함께 설치
```

<br/>

## 3. DVC Local 저장소 설정

```bash
$ mkdir dvc-tutorial
$ git init
$ dvc init
```

<br/>

## 4. DVC 기본명령

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

$ git commit -m "DVC initial commit"
$ git push origin main
```

<br/>

## 5. Remote Storage 설정

- data가 실제로 저장될 remote storage 설정 (Google Drive)
- Google Drive에서 새로운 폴더를 하나 생성한 뒤, url로부터 ID 부분을 복사

```bash
# dvc's default remote storage
$ dvc remote add -d storage gdrive://1tXxStPj2lmfnezLEr4CZ6Kt4POweYoFC

$ git add .dvc/config
$ git commit -m "add remote storage"
```

<br/>

## 6. 데이터 업로드/다운로드

```bash
# 데이터 업로드 (google login 인증 필요)
$ dvc push

# dvc cache 삭제
$ rm -rf .dvc/cache

# 데이터 삭제
$ rm dvc-tutorial/data/demo.txt

# 데이터 다운로드
$ dvc pull
```

<br/>

## 7. 데이터 버전 변경

```bash
$ vi demo.txt # 데이터 수정
$ dvc add demo.txt

$ git add demo.txt.dvc
$ git commit -m "Update demo.txt"
$ git push origin main
$ dvc push

# .dvc 파일을 이전 commit 버전으로 복구
$ git log --oneline
$ git checkout <commit-id>

# .dvc 내용을 보고 파일을 이전 버전으로 변경
$ dvc checkout
```