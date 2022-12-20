# MLflow

- **Tracking** : 모델의 하이퍼파라미터, 메트릭을 저장하는 중앙저장소를 제공, 모델 메타정보를 기록, 소스코드버전, 날짜, 패키징 모델파일 자체, 데이터 자체, 모델 description, tag 등을 기록한다.
- **Projects** : 모델 성능을 재현할 수 있도록 파라미터의 데이터 타입, PyTorch, Tensorflow, Docker, Conda 버전 등 모델에 의존성이 있는 모든 것들을 함께 패키징한다.
- **Models** : 모델이 어떤 형태로 개발되었든 상관없이 통일된 형태로 배포에 사용될 수 있도록 포맷화시킨다.
- **Model Registry** : 모델을 저장하는 저장소

<br/>

## Install

```bash
$ conda create -n mlflow-tutorial python=3.8.5
$ conda activate mlflow-tutorial

$ pip install mlflow
$ mlflow --version # 2.0.1
```

<br/>

## Tracking Server 띄우기

- tracking server를 띄울 때 `--backend-store-uri`, `--default-artifact-root` 옵션을 주지 않은 경우 `mlflow ui`를 실행한 디렉토리에 `mlruns` 디렉토리가 생성되고, 이곳에 실험 관련 데이터를 저장하게 된다.

```bash
# mlflow tracking server를 띄운다
# UI Dashboard의 default url은 http://localhost:5000
# 5000 port가 열려있는지 확인
# production 용으로는 mlflow ui 대신 mlflow server를 사용을 권장
$ mlflow ui -p 5001

# mlflow server는 worker를 여러 개 띄울 수 있다
# Prometheus가 metrics를 가져갈 수 있도록 endpoint를 제공하는 등의 추가적인 기능도 존재한다
$ mlflow server
```

<br/>

## Example Code1

- train_diabetes.py
  - 442명의 당뇨병 환자를 대상으로 나이, 성별, bmi 등의 독립 변수 10개를 가지고 1년 뒤 당뇨병의 진행률을 예측하는 문제
  - ElasticNet : Linear Regression + L1 Regularization + L2 Regularization
    - parameter
      - alpha : Regularization coefficient
      - l1_ratio : L1 Regularization과 L2 Regularization 비율
  - mlflow 코드
    - `mlflow.log_param`
    - `mlflow.log_metric`
    - `mlflow.log_model`
    - `mlflow.log_artifact`

```bash
# Example code download
# Linux 사용자
$ wget https://raw.githubusercontent.com/mlflow/mlflow/master/examples/sklearn_elasticnet_diabetes/linux/train_diabetes.py

# MacOS 사용자
$ wget https://raw.githubusercontent.com/mlflow/mlflow/master/examples/sklearn_elasticnet_diabetes/osx/train_diabetes.py

# ML 프로젝트가 백엔드 Tracking Server와 통신할 수 있도록 환경변수 설정
$ export MLFLOW_TRACKING_URI="http://127.0.0.1:5001"

# Example code run
$ python train_diabetes.py # mlartifacts 디렉토리에 아티팩트 저장소 생성
$ python train_diabetes.py 0.01 0.01
$ python train_diabetes.py 0.01 0.75
$ python train_diabetes.py 0.01 1.0
$ python train_diabetes.py 0.8 1.0
```

<br/>

## MLflow 데이터 저장 방식

```bash
$ cd mlruns/0
$ ls # 무작위 영숫자조합 디렉토리 이름은 mlflow의 run-id를 의미

$ cd <run-id>
$ ls # artifacts, metrics, params, tag 디렉토리와 mlflow run의 메타정보 파일도 존재
```

<br/>

## Model Serving

- `mlflow models serve`
- 모델 서빙은 해당 포트에서 REST API 형태로 `.predict()` 함수를 사용할 수 있는 것을 의미
- API에 요청을 보내기 위해 request body에 포함될 data의 형식을 알고 있어야 한다.

```bash
$ mlflow models serve --help

# $ brew install pyenv
# $ pip install virtualenv
# $ conda install python=3.8.10
$ mlflow models serve -m $(pwd)/mlartifacts/0/0b755d5dfa4f47c2a2387e731e57a101/artifacts/model -p 5005
$ 
```

<br/>

## Request

- API에 요청을 보내기 위해 request body에 포함될 data의 형식을 알고 있어야 한다.
- 예측값은 API의 response로 반환된다.

```python
data = load_diabetes()
print(data.feature_names)

df = pd.DataFrame(data.data)
print(df.head())
print(data.target[0])
```

- 127.0.0.1:5005 서버에서 제공하는 `POST /invocation` API 요청

```bash
$ curl -X POST -H "Content-Type:application/json" --data '{"columns":["age", "sex", "bmi", "bp", "s1", "s2", "s3", "s4", "s5", "s6"],"data":[[0.038076, 0.050680,  0.061696,  0.021872, -0.044223, -0.034821, -0.043401, -0.002592,  0.019908, -0.017646]]}' http://127.0.0.1:5005/invocations
```

<br/>

## Example Code2

- scikit-learn과 같은 패키지는 mlflow 레벨에서 `autolog`를 지원한다.
  - `mlflow.sklearn.autolog()`, `mlflow.xgboost.autolog()`
  - model의 parameter, metrics와 model artifacts를 사용자가 명시하지 않아도 자동으로 mlflow에 로깅
  - 추가적인 custom metric을 남기기 위해 `mlflow.log_metrics()`를 사용할 수 있다.

```bash
# Example code download
$ wget https://raw.githubusercontent.com/mlflow/mlflow/master/examples/sklearn_autolog/utils.py
$ wget https://raw.githubusercontent.com/mlflow/mlflow/master/examples/sklearn_autolog/pipeline.py
$ wget https://raw.githubusercontent.com/mlflow/mlflow/master/examples/xgboost/train.py

# Example code run
$ python pipeline.py
$ python train.py 		# xgboost==1.4.2 설치 필요
```