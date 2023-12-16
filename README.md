# Mini DevOps Project
![image](https://github.com/Saifweb/First_DevOps_Project/assets/98279240/b9de2540-d12f-4454-b287-ca35ceccf78a)

![image](https://github.com/Saifweb/First_DevOps_Project/assets/98279240/c3fa026b-e7c1-4d31-aed9-86ec48994b70)


## Overview

This project leverages several technologies for Continuous Integration (CI) and Deployment (CD), including Jenkins, Ansible, Minikube, and Kubernetes. Here's a brief overview of each component:

1. **Jenkins:**
    - Jenkins is used for CI/CD automation.
    - Installation: Follow [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-jenkins-on-ubuntu-20-04) to install Jenkins on a GCP Compute Engine instance.
    - Ensure Java is installed: `sudo apt install openjdk-11-jdk`.
    - Additional Resources:
        - [Jenkins Installation on GCP](https://www.youtube.com/watch?v=<video_id>).

2. **Ansible:**
    - Ansible is used for configuration management and deployment automation.
    - Installation: Refer to [this video](https://www.youtube.com/watch?v=xRMPKQweySE).
    - Instance Type: Verify a micro instance is used for Ansible.
    - Key Generation: Execute `sudo cat /var/lib/jenkins/secrets/initialAdminPassword` for the Ansible key.

3. **Webapp (Kubernetes):**
    - Kubernetes is used for container orchestration.
    - Instance Type: Verify a medium instance is used for the Webapp.
    - Docker Installation: Use the commands in the Dockerfile.
    - Minikube Installation: Refer to the commands in the Minikube file.
    - Minikube Explanation: Minikube facilitates local execution of Kubernetes.

## Work Steps

### 1. Link Jenkins with GitHub

- Establish a webhook for real-time communication between GitHub and Jenkins.
- Webhook Explanation: A mechanism for GitHub to notify Jenkins about repository events.
- CI Setup: Create a Dockerfile, commit it to GitHub, and configure Jenkins to trigger builds.

### 2. Send Docker File to Ansible

- Jenkins acts as a server, Ansible as a client.
- SSH Setup: Establish passwordless SSH between Jenkins and Ansible.
- Tutorial: [Configure SSH Passwordless Login](https://www.youtube.com/watch?v=<video_id>).

### 3. Build Docker Image and Tag

- Utilize Jenkins pipeline to connect to Ansible.
- Build Docker Image: Run `docker image build –t $JOB_NAME:v1.$BUILD_ID`.

### 4. Push Docker Images to Docker Hub

- Push the built Docker image to Docker Hub for versioning and tracking.

### 5. Deployment

- Establish SSH connection between Ansible and the Webapp server.
- Update Ansible's host file for best practices.

## Common Issues and Solutions

- **Mirrorlist Error:**
  - If Dockerfile fails, update it with the solution provided [here](https://stackoverflow.com/questions/70963985/error-failed-to-download-metadata-for-repo-appstream-cannot-prepare-internal).
## Jenkins Pipline
node{
    stage('Git checkout'){
        git branch: 'main', url: 'https://github.com/Saifweb/First_DevOps_Project.git'
    }
    stage('sending all files to Ansible server over ssh'){
        sshagent(['ansible_1']) {
            sh 'ssh -o StrictHostKeyChecking=no root@10.128.0.5'
            sh 'scp /var/lib/jenkins/workspace/pipeline_demo/* root@10.128.0.5:/home/saifbenhmida1/'

        }
    }
    stage('Docker image building'){
        sshagent(['ansible_1']) {
            sh '''
                ssh -o StrictHostKeyChecking=no root@10.128.0.5 "
                    cd /home/saifbenhmida1/
                    docker image build -t $JOB_NAME:v1.$BUILD_ID .
                "
            '''
        }
    }
    stage('Docker image tagging'){
        sshagent(['ansible_1']) {
            sh '''
                ssh -o StrictHostKeyChecking=no root@10.128.0.5 "
                    cd /home/saifbenhmida1/
                    docker image tag  $JOB_NAME:v1.$BUILD_ID saifbenhmida1420/$JOB_NAME:v1.$BUILD_ID
                    docker image tag  $JOB_NAME:v1.$BUILD_ID saifbenhmida1420/$JOB_NAME:latest
                "
            '''
        }
    }
    stage('push Docker images to Docker Hub'){
        sshagent(['ansible_1']) {
            withCredentials([string(credentialsId: 'dockerhub_passwd', variable: 'dockerhub_passwd')]) {
                sh '''
                    ssh -o StrictHostKeyChecking=no root@10.128.0.5 "
                        docker login -u saifbenhmida1420 -p ${dockerhub_passwd}
                        docker image push saifbenhmida1420/$JOB_NAME:v1.$BUILD_ID
                        docker image push saifbenhmida1420/$JOB_NAME:latest
                        docker image rm saifbenhmida1420/$JOB_NAME:v1.$BUILD_ID saifbenhmida1420/$JOB_NAME:latest
                    "
                '''
            }
        }
    }
    stage('push file from jenkins server to webapp server ( kubernetes server ) '){
        sshagent(['Kubernetes']) {
            sh 'ssh -o StrictHostKeyChecking=no root@10.128.0.6'
            sh 'scp /var/lib/jenkins/workspace/pipeline_demo/* root@10.128.0.6:/home/saifbenhmida1/'
        }
    }
    stage('Kubernetes Deployment using Ansible'){
        sshagent(['ansible_1']) {
            sh '''
                    ssh -o StrictHostKeyChecking=no root@10.128.0.5 "
                       cd /home/saifbenhmida1/
                       ansible-playbook ansible.yml
                    "
                '''
        }
    }
}
## Conclusion

Follow these steps to implement CI/CD with Jenkins, Ansible, and Kubernetes for efficient development and deployment workflows.
