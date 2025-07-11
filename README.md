
## MLOps Project

This project demonstrates an end-to-end data pipeline using Apache Airflow, Docker, and Kubernetes.

## Requirements
Virtual Environment
Python 3.8+
Streamlit
Apache Airflow
Docker & Docker Hub
Virtual Environment
Kubernetes
Kubectl

Set up Environment

sudo apt update && sudo apt upgrade -y
sudo apt install python3-pip -y


Create Python Virtual Environment & Activate it

python -m mlops_venv venv
source venv/bin/activate


pip3 install requirements.txt

Set up & Initialize the Airflow 

airflow version

export AIRFLOW_HOME=~/airflow

echo $AIRFLOW_HOME

### Create an Admin User for Airflow Web UI
1. Replace "auth_manager = airflow.api_fastapi.auth.managers.simple.simple_auth_manager.SimpleAuthManager" in airflow.cfg file with "auth_manager=airflow.providers.fab.auth_manager.fab_auth_manager.FabAuthManager". 
2. Or you can comment the earlier variable.


vim ~/airflow/airflow.cfg


Create Airflow DAGS Directory

mkdir -p ~/airflow/dags


Make sure the DAG is in the Airflow DAGS Directory

cp iris_model_pipeline_dag.py ~/airflow/dags/

ls ~/airflow/dags/


### Start the Airflow Scheduler, Api-Server, DB, Create Admin User - Starts all Components or Services of Airflow (For Dev Environment)

airflow standalone


On your browser: 

localhost:8080

Get Admin User Password: 

cat ~/airflow/simple_auth_manager_passwords.json.generated


### Build the DAG Pipeline

python ~/airflow/dags/iris_model_pipeline_dag.py


### On Airflow UI

1. Look for the DAG Pipeline named: iris_model_pipeline
2. Toggle the switch to "ON".
3. Click the "Trigger DAG" button (the play icon) to start run.
4. Monitor the run (in the "Graph" or "Grid" view). 

Once DAG Pipeline run is successful, the model artifact (.pkl file) is saved to location: /tmp 
ls -ld /tmp/

ls /tmp/iris_logistic_model.pkl


### Prediction based on Sample Inputs in Script

cat /tmp/iris_predictions.csv


## Load Model & Make Prediction on on Streamlit UI

streamlit run app.py


## Build Docker Image of the Trained Model & Its Dependencies In One Artifact 

### Copy ML Model Artifact (.pkl file) to PWD

cp /tmp/iris_logistic_model.pkl .

docker build -t ml-airflow-streamlit-app .


### Create PAT and Log in to Your DockerHub Account
If it fails, go to your DockerHub account and create a Personal Access Token (PAT) - This will be your login password

docker login -u iquantc



### Try Docker Build Again

docker build -t ml-airflow-streamlit-app .

docker run -p 8501:8501 ml-airflow-streamlit-app

### On your browser - Use Network URL or LocalHost

http://localhost:8501



## Tag Your Local Docker Image

docker tag ml-airflow-streamlit-app:latest iquantc/ml-airflow-streamlit-app:latest


## Push the Image to DockerHub

docker push iquantc/ml-airflow-streamlit-app:latest



## Deploy ML Model Docker Image to Kubernetes
Review manifest files

### Create Minikube cluster

minikube start --driver=docker


### Deploy the Kubernetes Manifest Files
Review the deployment manifest

kubectl apply -f deployment.yaml


### Check the Resources Created


kubectl get pods

kubectl get svc

minikube ip

### Open in Browser

http://<minikube-ip>:<NodePort>

## Clean up

minikube stop
minikube delete --all