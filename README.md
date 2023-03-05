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

# Installing AIRFLOW using HELM
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
# we are going to apply a ConfigMap resource in our kubernetes cluster
# This config map will be named as "airflow-variables"
# And it will contain our airflow environement variables
...
ExtraEnvFrom: |
    - configMapRef:
        name: 'airflow-variables'
# to verify if your variables were applied to the cluster you need to inspect from the CLI
# kubectl exec --stdin --tty <name-of-your-webserver-container> -n airflow -- /bin/bash
# then type: python (in your bash)
# from airflow.models import Variable
# Variable.get("my_s3_bucket")
...
# If you are getting the message "mkdir: cannot create directory ‘/bitnami/postgresql/data’: Permission denied"
# go to the file helm-charts/airflow/charts/postgresql/values.yaml
# And change the value volumePermissions.enabled to true
...
# Youl will need to create a git-credentials.yaml and apply to your cluster
gitSync:
  enabled: true
  repo: https://github.com/owshq-marlon/demo.git
  branch: master
  rev: HEAD
  depth: 1
  maxFailures: 0
  subPath: "helm-charts/airflow/dags"
  # if your repo needs a user name password
  # you can load them to a k8s secret like the one below
  #   ---
  #   apiVersion: v1
  #   kind: Secret
  #   metadata:
  #     name: git-credentials
  #   data:
  #     GIT_SYNC_USERNAME: <base64_encoded_git_username>
  #     GIT_SYNC_PASSWORD: <base64_encoded_git_password>
  # and specify the name of the secret below
  #
  credentialsSecret: git-credentials
...
enableBuiltInSecretEnvVars:
  AIRFLOW__CORE__ENABLE_XCOM_PICKLING: true
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
