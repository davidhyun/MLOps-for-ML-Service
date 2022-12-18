# MLOps 구성요소

## 1. 데이터

| 구분                   | 대표 S/W                                                     |
| ---------------------- | ------------------------------------------------------------ |
| 데이터 수집 파이프라인 | Apache Sqoop, Apache Flume, Apache Kafka, Flink, Spark Streaming, Airflow |
| 데이터 저장            | MySQL, Hadoop, Amazon S3, MinIO                              |
| 데이터 관리            | TFDV, DVC, Feast, Amundsen                                   |

<br/>

## 2. 모델

| 구분                    | 대표 S/W                                          |
| ----------------------- | ------------------------------------------------- |
| 모델 개발               | Jupyter Hub, Docker, Kubeflow, Optuna, Ray, katib |
| 모델 버전 관리          | Git, MLflow, Github Action, Jenkins               |
| 모델 학습 스케줄링 관리 | Grafana, Kubernetes                               |

<br/>

## 3. 서빙

| 구분              | 대표 S/W                                                     |
| ----------------- | ------------------------------------------------------------ |
| 모델 패키징       | Docker, Flask, FastAPI, BentoML, Kubeflow, TFServing, seldon-core |
| 서빙 모니터링     | Prometheus, Grafana, Thanos                                  |
| 파이프라인 매니징 | Kubeflow, argo workflows, Airflow                            |

<br/>

## ML Platform (SaaS)

위의 기능들을 ML Platform(SaaS) 형태로 함께 제공하는 `AWS SageMaker`, `GCP Vertex AI`, `Azure Machine Learning` 이 있다.
