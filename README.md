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

<img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_AWS/blob/main/Img/1%20create%20a%20new%20ecr%20repository%20for%20the%20app.PNG" width=800 />

### Creating Jenkins Credentials for Amazon ECR

1. Open your newly created Amazon ECR repository.
2. Click Push commands for the java-maven-app repository.
3. Copy the authentication command to obtain the ECR password:
   ```bash
   aws ecr get-login-password --region us-east-2 
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_AWS/blob/main/Img/1%20get%20ecr%20password.png" width=800 />
   
6. The username is AWS
7. Open Jenkins and go to Manage Jenkins.
8. Select Credentials from the menu.
9. Under (global) scope, click Add Credentials.
10. In the credentials form, configure the following: 
   * Kind: Username with password
   * Scope: Global
   * Username: AWS
   * ID: ecr-credentials
   * password: pasted password obtained in step 3

   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_AWS/blob/main/Img/2%20adding%20ecr%20credentials%20to%20jenkins.png" width=800 />
   
11. Click Create to save the credentials.

### Creating a Secret for AWS ECR
1. Verify that your EKS cluster is running:
   ```bash
   kubectl get nodes
   ```
   
2. Create a Docker registry secret for AWS ECR:
   ```bash
     kubectl create secret docker-registry aws-registry-key \
     --docker-server=734066168422.dkr.ecr.us-east-2.amazonaws.com \
     --docker-username=AWS \
     --docker-password=*<obtained when executing aws ecr get-login-password --region us-east-2 >*
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_AWS/blob/main/Img/3%20creating%20aws%20secret.png" width=800 />
   
3. Confirm that the secret was created:
   ```bash
     kubectl get secret
   ```
4. Use this secret in your Kubernetes deployment manifest to pull the image from ECR.
    
### Updating the Deployment YAML file
1. In your Java-Maven-App repository (forked from the previous demo6), create a new feature branch.
   
2. Open the deployment.yaml file and update the imagePullSecrets section to use the AWS ECR secret:
   ```bash
          spec:
            imagePullSecrets:
              - name: aws-registry-key
   ```

### Updating Jenkinsfile
1. Update the build stage in the Jenkinsfile to use ECR credentials:
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
2. Replace hardcoded values with environment variables for the Docker repository and ECR server:

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
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_AWS/blob/main/Img/2%20updating%20the%20build%20section%20with%20ecr%20credentials%20and%20repo.PNG" width=800 />
   
4. Update the environment block in the Jenkinsfile to define repository-related variables:
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
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_AWS/blob/main/Img/4%20splitting%20docker%20server%20and%20repo.PNG" width=800 />
   
5. Update the deployment.yaml file to reference the environment variables:
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

   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_AWS/blob/main/Img/3%20updating%20deployment%20with%20ecr%20repo.PNG" width=800 />

10. Commit your changes to the repository.
    
12. Run the Jenkins pipeline.
    
    <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_AWS/blob/main/Img/5%20pipeline%20ok%20with%20ECR.PNG" width=800 />
    
14. Verify that the pod is running in your cluster:
    
    ```bash
    kubectl get pods
    ```
    <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_AWS/blob/main/Img/5%20pod%20running%20with%20image%20from%20ECR.png" width=800 />
    
16. Confirm the Docker image is available in your ECR repository

    <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_AWS/blob/main/Img/6%20image%20available%20in%20ECR.PNG" width=800 />
