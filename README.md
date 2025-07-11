# AWS EKS
## 📦 Demo 7
This project is part of **Module 11: Kubernetes on AWS (EKS)** in the **TWN DevOps Bootcamp**.It shows how to set up a **complete CI/CD pipeline** using Jenkins to deploy an application to an Amazon EKS cluster, while pulling container images from **ECR**
**
[GitLab Repo](https://gitlab.com/devopsbootcamp4095512/devopsbootcamp_8_jenkins_pipeline/-/tree/complete_pipeline_EKS_ECR?ref_type=heads)

## 📌 Objective
Build a CI/CD pipeline that:
- Builds a Docker image of the app
- Pushes the image to a **ECR**
- Pulls the image into an **EKS deployment**
- Deploys the app to the EKS cluster automatically using Jenkins

## 🚀 Technologies Used

- **kubectl**: CLI to interact with Kubernetes.
- **eksctl**: CLI tool for creating and managing EKS clusters
- **DigitalOcean**: hosting Jenkins server.
- **AWS EKS**: AWS kubenertes cluster
- **AWS ECR**: Elastic Container Registry AWS
- **Java/Maven**: Java/Maven app from the previous demo when using the increment version
- **Docker**: Containarization.
   
## 📋 Prerequisites
- Ensure you have an AWS Account
- Kubectl is installed and configured to connect to the Kubernetes cluster.
- Jenkins server is running
- We are going to use the Java-Maven-App repository from the previous demo.
- The EKS cluster from the previous demo is running.
  
## 🎯 Features
- Create ECR repository
- Create ECR Credentials in Jenkins
- Adjust Building and Tagging
- Crete Secret for ECR
- Update Jenkinsfile
- Update the deploy step in the CI/CD to deploy the newly built application image from DockerHub to EKS cluster.
- Complete CI/CD pipeline:
  - CI step: Increment version
  - CI step: Build artifact for Java/Maven app
  - CI step: Build and push to DockerHub
  - CD step: Deploy new application to EKS
  - CD Step: Commit version update.
       
## 🏗 Project Architecture



## ⚙️ Project Configuration
### Creating ECR Repository
1. Go to yout AWS Console
2. Go to ECR
3. Create a New Private Repository
4.  

### Updating Deployment and Service YAML files
1. In your java-maven-app repository (forked from the previous demo6), create a new feature branch.
   
5. In deployment.yaml, update the configuration to use dynamic environment variables for the repo, image name, and app name:
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
    
9. In your Jenkinsfile, add the following environment variables under the environment block:
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
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_DockerHub/blob/main/Img/4%20defining%20app%20name%20as%20env.png" width=800 />
   
10. In the deploy stage of your Jenkins pipeline, use the envsubst command to substitute environment variables and apply the YAML files:

    <details><summary><strong> envsubst </strong></summary>
    The `envsubst` command replaces environment variables in a file or string with their current values. This is commonly used to inject runtime values into Kubernetes manifests or other configuration templates.
    </details>
```bash
    stage("deploy") {
    
                 steps {
    
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'AWS_jenkins_key'
                    ]]){
                        script {
                            //gv.deployApp()
                            echo "Deploying app"
                            echo 'Generating kubeconfig...'
                            sh '''
                                aws eks update-kubeconfig \
                                  --region $AWS_REGION \
                                  --name $CLUSTER_NAME \
                                  --kubeconfig $KUBECONFIG
                            '''
                            echo 'deploying docker image...'
                            sh 'kubectl get nodes'
    
                            sh '''
                                pwd
                                ls
                                cd java-maven-app/
                                envsubst < kubernetes/deployment.yaml | kubectl apply -f -
                                envsubst < kubernetes/service.yaml | kubectl apply -f -
                            '''
                        }
                    }
                }
            }
```
<img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_DockerHub/blob/main/Img/5%20passing%20the%20deployment%20file%20and%20srvice%20to%20pipeline.png" width=800 />


### Creating a Secret for DockerHub
In this demo, the ECR secret is created manually from the host machine and is not included in the pipeline. You must create the secret once for each namespace where it is needed.

1. Ensure your EKS cluster is running.
   ```bash
   kubectl get nodes
   ```
   <img src="" width=800 />
   
2. Create Secret
   ```bash
     kubectl create secret docker-registry my-registry-key \
     --docker-server=docker.io \
     --docker-username=xxxx \
     --docker-password=xxxxx
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_DockerHub/blob/main/Img/9%20cretaing%20secret%20locally%20as%20its%20one%20time%20in%20this%20case.png" width=800 />
   
3. Verify that the secret was created
   ```bash
     kubectl get secret
   ```
   
   
4. Add the secret to the deployment.yaml file

   ```bash
     spec:
        imagePullSecrets:
          - name: my-registry-key
     ```
  <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_DockerHub/blob/main/Img/secret.png" width=800 />
   
6. Commit the changes
7. Execute pipeline.
8. Check deployment and service
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_DockerHub/blob/main/Img/16%20checking%20deployment%20and%20servcie.png" width=800 />
