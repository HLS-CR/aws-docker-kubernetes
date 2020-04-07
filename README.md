## Installing Udagram Project - Microservices at Scale

The folder udacity-c3-feed-service-b contains identical code to udacity-c3-feed-service. It was added so that docker images show a different version from udacity-c3-feed-service. It is only needed for local testing purposes and so that we can run two docker containers simultaneously for A/B deployments.

# Tasks

## Prerequisites:

kops v1.16.0 or newer and kubectl v1.15.5 or newer are installed.

## Creating, provisioning and deleting AWS infrastructure with kops:

1. Create S3 Bucket:<br/>
   `aws s3api create-bucket --bucket lars-kops-state-store --region ap-southeast-1 --create-bucket-configuration LocationConstraint=ap-southeast-1`<br/>
2. Enable versioning for possible rollbacks:<br/>
   `aws s3api put-bucket-versioning --bucket lars-kops-state-store --versioning-configuration Status=Enabled`<br/>
3. Export KOPS_CLUSTER_NAME and KOPS_STATE_STORE:<br/>
   `export KOPS_CLUSTER_NAME=lars.k8s.local`<br/>
   `export KOPS_STATE_STORE=s3://lars-kops-state-store`<br/>
4. Create Kubernetes cluster on AWS:<br/>
   `kops create cluster --node-count=2 --node-size=t2.medium --zones=ap-southeast-1a`<br/>
5. Review and edit:<br/>
   `kops edit cluster`<br/>
6. Deploy Kubernetes cluster on AWS:<br/>
   `kops update cluster --name ${KOPS_CLUSTER_NAME} --yes`<br/>
7. Validate if Kubernetes cluster has been successfully created on AWS (takes 3-5 minutes on average, before commands shows EOF error):<br/>
   `kops validate cluster`<br/>

8. After successfully running / testing the project DELETE the created AWS infrastructure (this project COSTS $4-$6 per day to run on AWS):<br/>
   `kops delete cluster --name lars.k8s.local --yes`<br/>

## Setup Kubernetes environment:

1. Generate base64 encrypted values for aws credentials, Database User Name, and Database Password and insert the values into aws-secret.yaml and env-secret.yaml files.
2. Load secret files and check status:<br/>
   `kubectl apply -f aws-secret.yaml`<br/>
   `kubectl apply -f env-secret.yaml`<br/>
   `kubectl get secrets`<br/>
3. Load config map and check status:<br/>
   `kubectl apply -f env-configmap.yaml`<br/>
   `kubectl get configmap`<br/>
4. Apply Deployments and check status:<br/>
   `kubectl apply -f backend-feed-deployment.yaml`<br/>
   `kubectl apply -f backend-user-deployment.yaml`<br/>
   `kubectl apply -f frontend-deployment.yaml`<br/>
   `kubectl get pod -o wide`<br/>
5. Apply Services and check status:<br/>
   `kubectl apply -f backend-feed-service.yaml`<br/>
   `kubectl apply -f backend-user-service.yaml`<br/>
   `kubectl apply -f frontend-service.yaml`<br/>
   `kubectl get service`<br/>
6. Deploy reverse proxy and check status (only after successfully completing step 5):<br/>
   `kubectl apply -f reverseproxy-service.yaml`<br/>
   `kubectl apply -f reverseproxy-deployment.yaml`<br/>
   `kubectl get pod -o wide`<br/>
   `kubectl get service`<br/>
7. Perform port forwarding (each port need to be run in a separate terminal window and left running):<br/>
   `kubectl port-forward service/frontend 8100:8100`<br/>
   `kubectl port-forward service/reverseproxy 8080:8080`<br/>
8. Scale pod replica numbers up or down (down in this example):<br/>
   `kubectl scale --replicas=0 deployment/<your-pod>`<br/>

## Statuscheck and Debugging:

1. `kubectl get nodes`
2. `kubectl get pod --all-namespaces`
3. `kubectl describe pods <podname+id>`

## Continuous Integration / Continuous Development:

Travis CI is setup to monitor GitHub repo for updates and will automatically build and deploy the respective Docker containers if changes are detected.
For CI/CD via Travis CI to work, the DOCKER_PASSWORD and DOCKER_USERNAME environment variables must be set in the Travis CI account. Instructions for setup: https://docs.travis-ci.com/user/docker/.

## Screenshots:

![](course-03/exercises/screenshots/1.%20DockerHub.png)<br/>
![](course-03/exercises/screenshots/2.%20Local%20Containers.png)<br/>
![](course-03/exercises/screenshots/2.1%20Local%20Containers.png)<br/>
![](course-03/exercises/screenshots/2.2%20Local%20Containers.png)<br/>
![](course-03/exercises/screenshots/3.%20AWS%20Kubernetes%20Cluster.png)<br/>
![](course-03/exercises/screenshots/3.1%20AWS%20Kubernetes%20Cluster%20Running.png)<br/>
![](course-03/exercises/screenshots/3.2%20AWS%20Kubernetes%20Cluster%20Running.png)<br/>
![](course-03/exercises/screenshots/3.3%20AWS%20Kubernetes%20Cluster%20Running.png)<br/>
![](course-03/exercises/screenshots/4.%20Travis-CI.png)<br/>
![](course-03/exercises/screenshots/4.1%20Travis-CI%20Dev-Branch%20Docker%20Push.png)<br/>
![](course-03/exercises/screenshots/4.2%20Travis-CI%20Master-Branch%20Docker%20Push.png)<br/>
