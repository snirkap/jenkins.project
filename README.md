# jenkins project
Jenkins Automation for Docker Image Deployment,
This repository contains an automation script for Jenkins to facilitate the continuous integration and deployment process. It is designed to trigger a series of steps upon receiving a push event to a GitHub repository. The automation process involves cloning the code from GitHub, building a Docker image from a Dockerfile, uploading the image to Docker Hub, and finally deploying the image for the developer to test their code changes.
## Here you can see a diagram explaining the project:
![Screenshot 2023-12-03 102726](https://github.com/snirkap/jenkins.project/assets/120733215/5653c1fa-d537-4f41-9f87-48e130fb0b72)


## tutorial
### How to run the project:
1. first you need to go to jenkins to: 
    
   Dashboard > Manage Jenkins > Plugin Manager

   and install these plugins:

   CloudBees Docker Build and Publish plugin, Docker Commons, Docker Pipeline, Pipeline: GitHub Groovy Libraries, GitHub plugin, GitHub Pipeline for Blue Ocean
   Version, GitHub Branch Source, GitHub API Plugin, Git client, Git, Kubernetes, Kubernetes Client API Plugin, Kubernetes Credentials Plugin, Kubernetes 
   Credentials Provider.
   
2. you need to create credentials for docker hub and kubernetes.

   for docker hub you neet to go in jenkins to:
   
   Dashboard > Manage Jenkins > Credentials > System > Global credentials > Add Credential. 

   create a new credential and put user name and password to your docker hub user and calld to the credential "docker-hub-credentials" this should look like 
   this. 
   ![image](https://github.com/snirkap/jenkins.project/assets/120733215/2c7b237f-7d4a-4486-b0d4-0d3bae9c4b26)


   for kubernetes you need to:
   1. Login to Kubernetes master server.
   2. Navigate to ~/.kube directory and copy the content of config file
   3. Go to Jenkins Dashboard > Manage Jenkins > Manage Credentials > Jenkins > Global Credentials > Add Credential
   4. calld it " kubeconfig-credentials " 

   Here select Type as Kubernetes configuration (kubeconfig)
   Select Enter directly and paste the content of kubeconfig copied in step 2 in this box

   ![image](https://github.com/snirkap/jenkins.project/assets/120733215/94cdc179-5ac9-419b-b207-91b842f1af58)

3. copy the pipeline file to a new pipline in your jenkins and change the value of "def imageName" in the "Upload Image to Docker Hub" stage  to "your user name 
   in docker hub/the name of your repository:the version of the image that you want", and write the same in the value of " def image " in the " Deploy to 
   Kubernetes " stage.

4. start the pipeline and you will have a new Deployment in your Cluster.  


   

* index.html
  ```
  <!DOCTYPE html>
  <html>
  <body>
  <h1>hello from snir</h1>
  </body>
  </html>
this is a simple index.html file that contain random text for example: "hello from snir". this file is used by the httpd image and decides what it will display as soon as you put it in the appropriate folder.
* dockerfile
  ```
  FROM httpd:2.4
  COPY index.html /usr/local/apache2/htdocs/
  EXPOSE 80
### FROM httpd:2.4
The "FROM" instruction is used to specify the base image for this Docker image. In this case, the base image is "httpd:2.4", which means it's based on Apache HTTP Server version 2.4. The httpd image is an official image provided by the Docker team, and it sets up a basic Apache web server environment.
### COPY index.html /usr/local/apache2/htdocs/
The "COPY" instruction is used to copy files and directories from the local filesystem (the context of the build) into the Docker image. The format for this instruction is "COPY <src> <dest>", where <src> is the path of the file or directory on the local filesystem, and <dest> is the destination path within the Docker image.
In this Dockerfile, "COPY index.html /usr/local/apache2/htdocs/" is used to copy the index.html file into the "/usr/local/apache2/htdocs/" directory within the Docker image. The "/usr/local/apache2/htdocs/" directory is the default document root directory for the Apache HTTP Server.
* jenkinsfile
  ```
    pipeline {
    agent any
  
    stages {
      stage('Build Docker Image') {
                steps {
          script {
            node {
              git branch: 'main', url: 'https://github.com/snirkap/jenkins.project.git'
              def myEnv = docker.build 'snirkapah1/jenkins-project:1'
            }
          }
        }
      }
  
      stage('Upload Image to Docker Hub') {
        steps {
          script {
            docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
              def imageName = 'snirkapah1/jenkins-project:1'
              docker.image(imageName).push()
            }
          }
        }
      }
  
      stage('Deploy to Kubernetes') {
        environment {
          KUBECONFIG = credentials('kubeconfig-credentials')
        }
        steps {
          script {
            def deploymentName = "dep-jenkins-project"
            def containerName = "jenkins-project-con"
            def image = "snirkapah1/jenkins-project:1"
            def namespace = "default"
  
            sh "kubectl create deployment ${deploymentName} --image=${image} -n ${namespace}"
          }
        }
      }
    }
  }
### Pipeline Block
The "pipeline" block is the root block that defines the entire Jenkins pipeline. It specifies the agent to be used for executing the pipeline stages. In this case, "agent any" is used, which means the pipeline can run on any available agent (Jenkins slave).
### Stages Block
The "stages" block is used to define multiple stages that make up the pipeline. Each stage represents a logical part of the CI/CD process.

#### Stage: Build Docker Image
This stage is responsible for building a Docker image for the project.

Steps:
1. The "script" block is used to execute a Groovy script, which allows more complex logic to be executed in the pipeline.
2. Inside the "node" block, Jenkins will allocate a build executor (agent) to run the following steps.
3. The "git" step is used to checkout the code from the specified GitHub repository (https://github.com/snirkap/jenkins.project.git) and the 'main' branch.
4. The "docker.build" step is used to build the Docker image named 'snirkapah1/jenkins-project:1' from the current workspace directory.

#### Stage: Upload Image to Docker Hub
This stage is responsible for pushing the Docker image to Docker Hub.

Steps:
1. The "script" block is used again to execute a Groovy script.
2. "The docker.withRegistry" step configures the Docker registry to be used (in this case, Docker Hub - https://registry.hub.docker.com) and sets the credentials needed to push the image.
3. The "docker.image(imageName).push()" step pushes the previously built Docker image ('snirkapah1/jenkins-project:1') to Docker Hub.

#### Stage: Deploy to Kubernetes
This stage is responsible for deploying the Docker image to a Kubernetes cluster.

Steps:
1. The "environment" block allows you to define environment variables that will be available during the execution of the pipeline.
2. The "KUBECONFIG" environment variable is set to the Kubernetes configuration credentials, which are likely stored in Jenkins as 'kubeconfig-credentials'.
3. The "script" block is used again for executing a Groovy script.
4. The "sh" step executes a shell command to deploy the Docker image to Kubernetes. It uses the Kubernetes "kubectl" command to create a deployment named 'dep-jenkins-project' with the image 'snirkapah1/jenkins-project:1' in the 'default' namespace.
#### Summary
In summary, this Jenkinsfile defines a declarative pipeline with three stages: building a Docker image, pushing the image to Docker Hub, and deploying the image to a Kubernetes cluster. The pipeline is designed to run on any available agent and includes logic to fetch code from a GitHub repository, build and push the Docker image to Docker Hub, and finally deploy it to Kubernetes.
