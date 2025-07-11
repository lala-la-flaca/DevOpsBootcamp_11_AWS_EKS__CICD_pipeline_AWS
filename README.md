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
- Create k8 manifest files for Deployment and Service configuration.
- Update the deploy step in the CI/CD to deploy the newly built application image from DockerHub to EKS cluster.
- Complete CI/CD pipeline:
  - CI step: Increment version
  - CI step: Build artifact for Java/Maven app
  - CI step: Build and push to DockerHub
  - CD step: Deploy new application to EKS
  - CD Step: Commit version update.
       
## 🏗 Project Architecture



## ⚙️ Project Configuration
### Creating Deployment and Service YAML files
1. In your java-maven-app repository (forked from the Java Increment Version Demo), create a new feature branch.
   
2. Create a new directory named kubernetes.
   
3. In the kubernetes directory, create two files:
   *  deployment.yaml
   *  service.yaml

   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_DockerHub/blob/main/Img/1%20create%20a%20new%20kubernetes%20folder%20and%20deployment%20and%20servicel%20yaml.png" width=800 />
   
4. Copy the baseline configurations into both files.
   
5. In deployment.yaml, update the configuration to use dynamic environment variables for the image name and app name:
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
                image: lala011/demo-app:$IMAGE_NAME
                imagePullPolicy: Always
                ports:
                  - containerPort: 8080
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_DockerHub/blob/main/Img/2%20setting%20a%20dynamic%20container%20image.png" width=800 />
   
8. In service.yaml, replace static values with the $APP_NAME environment variable:
   ```bash
       apiVersion: v1
       kind: Service
       metadata:
         name: $APP_NAME
       spec:
         selector:
           app: $APP_NAME
         ports:
           - protocol: TCP
             port: 80
             targetPort: 8080
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_DockerHub/blob/main/Img/3%20Replacing%20labels%20and%20app%20name%20with%20ENV.png" width=800 />
   
9. In your Jenkinsfile, add the following environment variables under the environment block:
   ```bash
      environment {
          KUBECONFIG = "${env.WORKSPACE}/kubeconfig"
          AWS_REGION = 'us-east-2'
          CLUSTER_NAME = 'demo-cluster'
          APP_NAME = 'java-maven-app'
  
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

### Installing Gettext-base on Jenkins
1. SSH into the Jenkins server (hosted on your DigitalOcean droplet):
   ```bash
      ssh root@198.199.70.18
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_DockerHub/blob/main/Img/6%20ssh%20to%20jenkins.png" width=800 />
   
3. Access to the Jenkins container as the root user.
   ```bash
      docker exec -u 0 -it 6db6fdd7ed8f bash
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_DockerHub/blob/main/Img/7%20entering%20jenkins%20ocntainer%20as%20root.png" width=800/>
   
5. Install gettext-base
   ```bash
     apt-get update
     apt-get install gettext-base
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_11_AWS_EKS__CICD_pipeline_DockerHub/blob/main/Img/8%20installing%20gettext-base%20envsubst.png" width=800 />
   
6. Exit the container

### Creating a Secret for DockerHub
In this demo, the Docker Hub secret is created manually from the host machine and is not included in the pipeline. You must create the secret once for each namespace where it is needed.

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
