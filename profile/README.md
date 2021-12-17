# Setup Instructions
## Installing the project in GKE
### Prerequisites
#### Google Cloud Platform (GCP) & Google Kubernetes Engine (GKE):
* Create Google Cloud Platform user - https://cloud.google.com
* Create a cloud project.
* Create a cluster in Google Kubernetes Engine - GKE Standard Cluster.
#### Organisation/repository secrets:
* Add dockerhub username and accesstoken in secrets as `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`
* Add GCP Project_ID in secrets as `GKE_PROJECT`
* Get `GKE_SA_KEY` and add as a secret - https://docs.github.com/en/actions/deployment/deploying-to-your-cloud-provider/deploying-to-google-kubernetes-engine
#### Access to gcloud CLI either in GCP or locally:
* (optional) install gcloud SDK locally - https://cloud.google.com/sdk/docs/install
* Log in using the credentials created ealier - https://cloud.google.com/sdk/gcloud/reference/auth/login
### Add Helm charts & create namespace
```bash 
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```
```bash
kubectl create ns rabbits
```
### Install Prometheus
#### Install using Helm:
```bash
helm install prometheus -f https://raw.githubusercontent.com/DM885/PrometheusK8S/main/prometheus-values-GKE.yaml prometheus-community/kube-prometheus-stack
```
#### Prometheus rule-set:
```bash
kubectl apply -f https://raw.githubusercontent.com/DM885/PrometheusService/main/prometheus-roles.yaml
```
#### RabbitMQ service monitor:
```bash
kubectl apply -f https://raw.githubusercontent.com/DM885/PrometheusService/main/rabbitmq-servicemonitor.yaml
```
### Install MySQL
#### Secrets
```bash
kubectl apply -f https://raw.githubusercontent.com/DM885/MySQLK8S/main/mysql-secrets.yaml
kubectl -n rabbits apply -f https://raw.githubusercontent.com/DM885/MySQLK8S/main/mysql-secrets.yaml
```
#### Storage
```bash
kubectl apply -f https://raw.githubusercontent.com/DM885/MySQLK8S/main/mysql-sc.yaml
kubectl apply -f https://raw.githubusercontent.com/DM885/MySQLK8S/main/mysql-pv.yaml
kubectl apply -f https://raw.githubusercontent.com/DM885/MySQLK8S/main/mysql-pvc-gke.yaml
```
#### Install using Helm
```bash
helm upgrade mysql -f https://raw.githubusercontent.com/DM885/MySQLK8S/main/mysql-values.yaml bitnami/mysql --set metrics.enabled=true --set serviceMonitor.enabled=true
```
### Install RabbitMQ:
```bash
kubectl -n rabbits apply -f https://raw.githubusercontent.com/DM885/RabbitMQK8S/main/rabbit-rbac.yaml
kubectl -n rabbits apply -f https://raw.githubusercontent.com/DM885/RabbitMQK8S/main/rabbit-configmap.yaml
kubectl -n rabbits apply -f https://raw.githubusercontent.com/DM885/RabbitMQK8S/main/rabbit-secret.yaml
kubectl -n rabbits apply -f https://raw.githubusercontent.com/DM885/RabbitMQK8S/main/rabbit-statefulset-gke.yaml
```
### Install Microservices:
kubectl -n rabbits apply -f https://raw.githubusercontent.com/DM885/GatewayService/main/deployment.yaml
kubectl -n rabbits apply -f https://raw.githubusercontent.com/DM885/CRUDservice/main/deployment.yaml
kubectl -n rabbits apply -f https://raw.githubusercontent.com/DM885/SolverInfoService/main/deployment.yaml
kubectl -n rabbits apply -f https://raw.githubusercontent.com/DM885/AuthenticationService/main/deployment.yaml
kubectl -n rabbits apply -f https://raw.githubusercontent.com/DM885/LoggingService/main/deployment.yaml
kubectl -n rabbits apply -f https://raw.githubusercontent.com/DM885/JobService/main/deployment.yaml
kubectl -n rabbits apply -f https://raw.githubusercontent.com/DM885/MiniZincService/main/deployment.yaml

