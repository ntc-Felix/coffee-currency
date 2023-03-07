# Intro

Para reproduzir esta POC, precisaremos primeiramente de acesso a um cluster K8S.
No meu caso, estarei utilizando o Minikube para simular um cluster localmente.

# Principais Componentes
- Apache Spark
- Trino  (aka PRESTO)
- Delta Lake
- MinIO
- Apache Superset
- Apache Airflow

# Minikube deployment

```sh
minikube start --nodes 3 --memory 16384 --cpus 2 --driver=virtualbox --disk-size 100GB
```
# Levantando recursos no cluster

# Apache Airflow
- Setting up helm
```sh
helm repo add apache-airflow https://airflow.apache.org/
helm pull apache-airflow/airflow
tar zxvf airflow-1.6.0.tgz
mv airflow helm-charts/
rm -rf airflow-1.6.0.tgz
```

- Changing helm values
```sh
gitSync:
  enabled: true
  repo: https://github.com/owshq-marlon/demo.git
  branch: master
  rev: HEAD
  depth: 1
  maxFailures: 0
  subPath: ""
```

```sh
helm install -n airflow airflow helm-charts/airflow --debug
```

# MinIO
```sh
helm repo add minio https://charts.min.io/ 
helm pull minio/minio
tar zxvf minio-4.0.14.tgz
mv minio helm-charts/
```

```sh
helm install minio helm-charts/minio -n minio --create-namespace --debug
```

# Trino
```sh
helm repo add trino https://trinodb.github.io/charts
helm pull trino/trino
tar zxvf trino-0.9.0.tgz
mv trino helm-charts/
```
```sh
helm install trino helm-charts/trino -n trino --create-namespace --debug
```

# Apache Superset
```sh
helm repo add superset https://apache.github.io/superset
helm pull superset/tsupersetrino
tar zxvf superset-0.8.7.tgz
mv superset helm-charts/
```

```sh
helm install superset helm-charts/superset -n superset --create-namespace --debug
```