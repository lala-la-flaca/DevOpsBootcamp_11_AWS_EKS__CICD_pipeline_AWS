# AWS EKS
## 📦 Demo 7
This project is part of **Module 11: Kubernetes on AWS (EKS)** in the **TWN DevOps Bootcamp**.It shows how to set up a **complete CI/CD pipeline** that builds, pushes, and deploys an application container to an **Amazon EKS cluster** using images stored in **Amazon ECR**, using **Jenkins**.
**
[GitLab Repo](https://gitlab.com/devopsbootcamp4095512/devopsbootcamp_8_jenkins_pipeline/-/tree/complete_pipeline_EKS_ECR/java-maven-app?ref_type=heads)

## 📌 Objective
Build a CI/CD pipeline that:
- Builds a Docker image of the app
- Pushes the image to a **ECR**
- Pulls the image into an **EKS deployment**
- Deploys the app to the EKS cluster automatically using Jenkins

## 🚀 Technologies Used

- **kubectl**: CLI to manage Kubernetes clusters and deploy workloads
- **eksctl**: CLI tool for creating and managing EKS clusters
- **DigitalOcean**: hosting Jenkins server.
- **AWS EKS**: Managed Kubernetes control plane for orchestrating containers
- **AWS ECR**: AWS container registry to store and version Docker images
- **Java/Maven**: Java/Maven app from the previous demo when using the increment version
- **Docker**: Builds application containers from source.
- **Jenkins**: CI/CD server to automate build, push, and deploy stages.
   
## 📋 Prerequisites
- Ensure you have an AWS Account
- Kubectl is installed and configured to connect to the Kubernetes cluster.
- Jenkins server is running
- We are going to use the Java-Maven-App repository from the previous demo.
- The EKS cluster from the previous demo is running.
  
## 🎯 Features
- Create an ECR repository.
- Create ECR Credentials in Jenkins
- Adjust Building and Tagging
- Crete Secret for ECR
- Update Jenkinsfile
- Update the deploy step in the CI/CD to deploy the newly built application image from ECR to the EKS cluster.
- Complete CI/CD pipeline:
     - CI step: Increment version
     - CI step: Build artifact for Java/Maven app
     - CI step: Build and push to DockerHub
     - CD step: Deploy new application to EKS
     - CD Step: Commit version update.
       
## 🏗 Project Architecture



## ⚙️ Project Configuration
### Creating ECR Repository
1. Go to your AWS Console
2. Go to ECR
3. Create a New Private Repository
4. Name the repository java-maven-app
5. Click Create Repository.

### Creating Username and Password for the ECR
1. Go to the newly created repository
2. Click Push commands for java-maven-app
3. Get the login and password.
4. Execute on your command line to obtain the password of the ECR repository
   ```bash
   aws ecr get-login-password --region us-east-2 
   ```
5. The username is AWS
6. Go to Jenkins and click  Manage Jenkins
7. Select Credentials
8. Create Global credentials
   Kind: Username with password
   Scope: Global
   Username: AWS
   ID: ecr-credentials
   password: *<ECR password>*
9. Click Create

### Creating a Secret for AWS ECR
1. Ensure your EKS cluster is running.
   ```bash
   kubectl get nodes
   ```
   <img src="" width=800 />
   
2. Create Secret
   ```bash
     kubectl create secret docker-registry aws-registry-key \
     --docker-server=734066168422.dkr.ecr.us-east-2.amazonaws.com \
     --docker-username=AWS \
     --docker-password=*<obtained when executing aws ecr get-login-password --region us-east-2 >*
   ```
   <img src="" width=800 />
   
3. Verify that the secret was created
   ```bash
     kubectl get secret
   ```
4. The secret is going to be used in the deployment configuration file to fetch the image.
    
### Updating the Deployment YAML file
1. In your Java-Maven-App repository (forked from the previous demo6), create a new feature branch.
   
2. In deployment.yaml, update the imagePullSecret to use the AWS registry
   ```bash
          spec:
            imagePullSecrets:
              - name: aws-registry-key
   ```
   <img src="" width=800 />

### Updating Jenkinsfile
1. In the build stage, update the Jenkinsfile to use the ECR credentials
   ```bash
      stage("build image") {
               steps {
                   script {
                       echo "building the docker image..."
                       withCredentials([usernamePassword(credentialsId: 'ecr-credentials', passwordVariable: 'PWD', usernameVariable: 'USER')]){
                               sh """
                                   cd java-maven-app/
                               """
                               }
                   }
               }
            }
   ```
2. Set the New repository on the Docker build command and extract the hardcoded string to an ENV variable named $DOCKER_REPO and add the server of the repository using the $DOCKER_REPO_SERVER 
   ```bash
        stage("build image") {
                  steps {
                      script {
                          echo "building the docker image..."
                          withCredentials([usernamePassword(credentialsId: 'ecr-credentials', passwordVariable: 'PWD', usernameVariable: 'USER')]){
                                  sh """
                                      cd java-maven-app/
                                      docker build -t $DOCKER_REPO:${IMAGE_NAME} .
                                      echo $PWD | docker login -u $USER --password-stdin $DOCKER_REPO_SERVER
                                      docker push $DOCKER_REPO:${IMAGE_NAME}
                                  """
                                  }
                      }
                  }
               }
   ```
3. Update the environment variables section of the Jenkinsfile:
   ```bash
         environment {
              KUBECONFIG = "${env.WORKSPACE}/kubeconfig"
              AWS_REGION = 'us-east-2'
              CLUSTER_NAME = 'demo-cluster'
              APP_NAME = 'java-maven-app'
              DOCKER_REPO_SERVER = '734066168422.dkr.ecr.us-east-2.amazonaws.com'
              DOCKER_REPO = "${DOCKER_REPO_SERVER}/java-maven-app"
          }
   ```
   <img src="" width=800 />
   
4. Update the deployment.yaml to access the repository's environment variable.
   ```bash
         apiVersion: apps/v1
         kind: Deployment
         metadata:
           name: $APP_NAME
           labels:
             app: $APP_NAME
         spec:
           replicas: 2
           selector:
             matchLabels:
               app: $APP_NAME
           template:
             metadata:
               labels:
                 app: $APP_NAME
             spec:
               imagePullSecrets:
                 - name: aws-registry-key
               containers:
                 - name: $APP_NAME
                   image: $DOCKER_REPO:$IMAGE_NAME
                   imagePullPolicy: Always
                   ports:
                     - containerPort: 8080    
   ```

   <img src="" width=800 />
10. Commit changes
11. Execute the pipeline
12. Check the repository
    
   
