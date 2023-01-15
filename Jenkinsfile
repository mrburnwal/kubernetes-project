node{
    stage('git checkout') {
        git 'https://github.com/mrburnwal/kubernetes-project.git'
    }
    
    stage('sending Dockerfile to Ansible-Server'){
        sshagent(['ansible-cred']) {
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.39.139'
            sh 'scp /var/lib/jenkins/workspace/kubernets-deploy/* ubuntu@172.31.39.139:/home/ubuntu'
        }
    }
    
    stage('Docker Image Build'){
        sshagent(['ansible-cred']){
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.39.139 cd /home/ubuntu'
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.39.139 docker build -t $JOB_NAME:v1.$BUILD_ID .'
        }
    }
    
    stage('Docker Image Tagging'){
        sshagent(['ansible-cred']){
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.39.139 docker image tag $JOB_NAME:v1.$BUILD_ID mrburnwal/$JOB_NAME:v1.$BUILD_ID'
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.39.139 docker image tag $JOB_NAME:v1.$BUILD_ID mrburnwal/$JOB_NAME:latest'
        }
    }
    
    stage('Push Docker Image to DockerHub'){
        sshagent(['ansible-cred']){
            withCredentials([string(credentialsId: 'dockerhub_pass', variable: 'dockerhub_pass')]) {
                sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.39.139 docker login -u mrburnwal -p $dockerhub_pass'
                sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.39.139 docker push mrburnwal/$JOB_NAME:v1.$BUILD_ID'
                sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.39.139 docker image push mrburnwal/$JOB_NAME:latest'
                sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.39.139 docker image rm mrburnwal/$JOB_NAME:v1.$BUILD_ID mrburnwal/$JOB_NAME:latest $JOB_NAME:v1.$BUILD_ID'
            }
        }
    }
    
    stage('File copied from Jenkins server to Kubernetes server'){
        sshagent(['kubernetes-server']) {
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.4.73'
            sh 'scp /var/lib/jenkins/workspace/kubernets-deploy/* ubuntu@172.31.4.73:/home/ubuntu'
        }
    }
    
    stage('run asible playbook to deploy kubernetes nodes'){
        sshagent(['ansible-cred']){
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.39.139 cd /home/ubuntu'
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.39.139 ansible-playbook ansible.yml'
        }
    }
}
