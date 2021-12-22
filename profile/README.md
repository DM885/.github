# Setup Instructions
## Installing the project in GKE
### Prerequisites
#### Google Cloud Platform (GCP) & Google Kubernetes Engine (GKE):
* Create Google Cloud Platform user - https://cloud.google.com
* Create a cloud project.
* Create a cluster in Google Kubernetes Engine - GKE Standard Cluster.
#### Organisation/repository secrets:
* Add personal access token from any one with reading privileges for the private repositories. Call it `AT`
* Add dockerhub username and accesstoken in secrets as `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN`
* Add GCP Project_ID in secrets as `GKE_PROJECT`
* Get `GKE_SA_KEY` and add as a secret - https://docs.github.com/en/actions/deployment/deploying-to-your-cloud-provider/deploying-to-google-kubernetes-engine
#### Access to gcloud CLI either in GCP or locally:
* (optional) install gcloud SDK locally - https://cloud.google.com/sdk/docs/install
* Log in using the credentials created ealier - https://cloud.google.com/sdk/gcloud/reference/auth/login

Get credentials in GCP shell for the cluster to use kubectl in the shell:
```
gcloud container clusters get-credentials buildcluster --region=europe-north1
```

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
helm install prometheus -f /PrometheusK8S/prometheus-values-GKE.yaml prometheus-community/kube-prometheus-stack
```
#### Prometheus rule-set:
```bash
kubectl apply -f /PrometheusService/prometheus-roles.yaml
```
#### RabbitMQ service monitor:
```bash
kubectl apply -f /PrometheusService/rabbitmq-servicemonitor.yaml
```
### Install MySQL
#### Secrets
```bash
kubectl apply -f /MySQLK8S/mysql-secrets.yaml
kubectl -n rabbits apply -f /MySQLK8S/mysql-secrets.yaml
kubectl -n rabbits apply -f /MySQLK8S/auth-secrets.yaml
```
#### Storage
```bash
kubectl apply -f /MySQLK8S/mysql-sc.yaml
kubectl apply -f /MySQLK8S/mysql-pv.yaml
kubectl apply -f /MySQLK8S/mysql-pvc-gke.yaml
```
#### Install using Helm
```bash
helm install mysql -f /MySQLK8S/mysql-values-gke.yaml bitnami/mysql
```
### Install RabbitMQ:
```bash
kubectl -n rabbits apply -f /RabbitMQK8S/rabbit-rbac.yaml
kubectl -n rabbits apply -f /RabbitMQK8S/rabbit-configmap.yaml
kubectl -n rabbits apply -f /RabbitMQK8S/rabbit-secret.yaml
kubectl -n rabbits apply -f /RabbitMQK8S/rabbit-statefulset-gke.yaml
```
Once the pods have initialized. Configure rabbitmq to use queue mirroring and set a TTL for the messages in the queue:
```bash
kubectl -n rabbits exec -it pod/rabbitmq-0 bash
```
Once inside:
```bash
rabbitmqctl set_policy TTL ".*" '{"message-ttl":60000}' --apply-to queues
```
```bash
rabbitmqctl set_policy ha-fed \
    ".*" '{"federation-upstream-set":"all", "ha-sync-mode":"automatic", "ha-mode":"nodes", "ha-params":["rabbit@rabbitmq-0.rabbitmq.rabbits.svc.cluster.local","rabbit@rabbitmq-1.rabbitmq.rabbits.svc.cluster.local"]}' \
    --priority 1 \
    --apply-to queues
```
Exit the pod and continue.
### Grafana Dashboard:
#### Dashboards -> Import, and import these two ID's:
```
14057
```
```
10991
```

### Install Microservices:
```bash
kubectl -n rabbits apply -f /GatewayService/deployment.yaml
kubectl -n rabbits apply -f /CRUDservice/deployment.yaml
kubectl -n rabbits apply -f /SolverInfoService/deployment.yaml
kubectl -n rabbits apply -f /AuthenticationService/deployment.yaml
kubectl -n rabbits apply -f /LoggingService/deployment.yaml
kubectl -n rabbits apply -f /JobService/deployment.yaml
kubectl -n rabbits apply -f /MiniZincService/deployment.yaml
```

## Setup the pipeline in a Github repository
### Prerequisites
* The organisation/repository secrets from above.
* `test_services.sh` in the root folder of the repository containing commands to add the services which the integrations tests needs to succeed.
* `deployment.yaml` in the root folder of the repository, setup correctly to be deployed to kubernetes. Must have unique labels/names in the file. 

### Github Actions CICD pipeline:
#### The files for the pipeline will be in the folder "Pipeline"
* Create a folder `.github/` in the root folder of the repository.
* Create a folder `workflows/` in the `.github/` folder.
* Add the bash files `k3s-setup.sh` and `k3s-wait.sh` in `.github/`
* Add the yaml files `main.yaml` and `developer.yaml` in `.github/workflows/`
* Go into the `main.yaml` file and change the env variable `DEPLOYMENT_NAME`

* Lastly, enable the CICD pipeline by setting the env variable `CICD_TOGGLE`to `true` in `main.yaml
## UI
ENV variables
```
apiURL: The URL to the API, defaults to either localhost or our api if not set.
```
### Setup
Sign in as an administrator and add the following solvers.

| Solver name | Image             |
|-------------|-------------------|
| gecode      | sejkom/gecode     |
| chuffed     | sejkom/chuffed    |


### Deployment Guide

#### Deploy to GitHub Pages

Push this folder to you GitHub acount. This GiitHub action will trigger every time a change is pushed to the main branch. It will build the React app and deplay the content of the builld directory to 'gh-pages' branch.

a GITHUB_TOKEN is needed in ${{secrets.GITHUB_TOKEN }}. Github wil normally automatiically create a token secret to use in the workflow. It comes with write access to the reporsitory. And therforere it is allowed to update the 'gh-pages' branch.

#### Setup GitHub pages

Now we need to enable GitHubPages. Click on settings in the menu. Choose 'Pages'. The build files are pushed to 'gh-pages' choose this branch as sources. Click on the save button

#### Setup homepage

Open up the source code and in the package.json add a key-value pair. Insert and replace this: "homepage": "https://.github.io//",
