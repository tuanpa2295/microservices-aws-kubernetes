
# Coworking Analytics Deployment Documentation
## Overview
This documentation provides guidance on deploying changes to the Udacity "Coworking Analytics" project. It aims to help experienced software developers understand the technologies and tools involved in the build and deployment process, as well as provide insight into how to release new builds.

## Tech Stack

The Coworking Analytics project is built using modern technologies and tools commonly used in web development. Here's an overview of the key components:

**Server:** Python

**Database:** Postgres

**Cloud:** AWS Cloudwatch, EKS, IAM, CodeBuild, ECR

**Containerization:** Docker


## Setup

Set up locally
```bash
export DB_USERNAME=myuser
export DB_PASSWORD=${POSTGRES_PASSWORD}
export DB_HOST=127.0.0.1
export DB_PORT=5433
export DB_NAME=mydatabase


python app.py


# Set up port forwarding
kubectl port-forward --namespace default svc/<SERVICE_NAME>-postgresql 5433:5432 &
# Export the password. Replace 
export POSTGRES_PASSWORD=$(kubectl get secret --namespace default <SERVICE_NAME>-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)


# You may run these commands in another terminal window:
curl <BASE_URL>/api/reports/daily_usage
curl <BASE_URL>/api/reports/user_visits

```

Update kubeconfig ~/.kube/config to connect to cluster
```bash
aws configure --profile=<PROFILE-NAME>

export AWS_PROFILE=<PROFILE-NAME>

aws sts get-caller-identity

aws eks --region us-east-1 update-kubeconfig --name my-cluster

# Find the service name of postgresql by listing your services
kubectl get svc

# Set up port-forwarding for the postgresql service
kubectl port-forward svc/postgresql-service 5433:5432 &


```
    
Verify and copy the context name

```
kubectl config current-context

```

Output should be similar to: 

```
arn:aws:eks:us-west-2:411764878390902:cluster/my-cluster

```
## Deployment Process

### Local Development Environment

**1. Setup Development Environment:** Clone the project repository and ensure that Docker and other necessary development tools are installed on your local machine.

**2. Run Docker Containers:** Use Docker CLI to run containers for the backend, database locally (or can connect to ).

**3. Access the Application:** Once the containers are up and running, access the application through a web browser or API client using the specified port: `5153`.

``` 
# Update the local package index with the latest packages from the repositories
apt update

# Install a couple of packages to successfully install postgresql server locally
apt install build-essential libpq-dev

# Update python modules to successfully build the required modules
pip install --upgrade pip setuptools wheel

pip install -r requirements.txt

```

### Deployment CI/CD
**1. Version Control:** Ensure that all changes to the project are committed and pushed to the version control Github.

**2. CI Pipeline:** The CI pipeline is triggered whenever changes are pushed to the repository. It performs tasks such as building Docker images,and generating artifacts and pushing image artifact to ECR.

**3. Artifact Repository:** Artifacts generated during the CI process, such as Docker images, are stored in a repository ECR at `741692117543.dkr.ecr.us-east-1.amazonaws.com/coworking`.


**4. CD Pipeline:** The CD pipeline deploys changes to the target environment by running when we see a new image artiffact pushed to ECR.

```bash
  kubetctl set image deployments/coworking coworking=741692117543.dkr.ecr.us-east-1.amazonaws.com/coworking:<IMAGE-TAG>
```


## Lessons Learned

### Memory and CPU allocation:
Pods CPU and Memory shoule have resource requests & limits to prevent over-consumption and to optimize reouces.

Here is an example:
```
...
resources:
    limits:
        memory: 200Mi
        cpu: 250m
    requests:
        memory: 150Mi
        cpu: 200m    
...
```

### EC2 Instance Type used:

For EKS Nodes, a `t3.medium` can be used for controlplane node. `t2.medium` for workernode. In case of scale up, multiple `t2.medium` nodes can join the cluster & node group.

### Cost savings:

We can set up logging & monitoring to prevent over consumption of resources. Also we can set up cluster using internal internet traffic to reduce price & not use S3 to store artifact.
