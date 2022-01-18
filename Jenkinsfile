pipeline {
    agent any
    environment {
        registry = "905140238863.dkr.ecr.ap-southeast-1.amazonaws.com/devops2-test"
    }
    stages {
        stage('Cloning Git') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/huor-panha/ams-docker.git']]])
            }
        }
//         stage('Docker build') {
//             steps {
//                 script {
//                   dockerImage = docker.build registry
//                 }
//             }
//         }
//         stage('Docker push') {
//             steps {
//                 script {
//                   sh 'aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 905140238863.dkr.ecr.ap-southeast-1.amazonaws.com'
//                   sh 'docker push 905140238863.dkr.ecr.ap-southeast-1.amazonaws.com/devops2-test:latest'
//                 }
//             }
//         }
//         // Stopping Docker containers for cleaner Docker run
//         stage('Stop previous containers') {
//              steps {
//                 sh 'docker ps -f name=ams-docker -q | xargs --no-run-if-empty docker container stop'
//                 sh 'docker container ls -a -fname=ams-docker -q | xargs -r docker container rm'
//              }
//         }
//         stage('Docker Run') {
//             steps{
//                 script {
//                     sh 'docker run -d -p 80:80 -e APP_KEY="base64:3ilviXqB9u6DX1NRcyWGJ+sjySF+H18CPDGb3+IVwMQ=" --rm --name ams-docker 905140238863.dkr.ecr.ap-southeast-1.amazonaws.com/devops2-test:latest'
//                 }
//             }
//         }
        stage('Run commands on remote server') {
            steps { 
                script {
                    def remote = [:]
                    remote.user = 'ubuntu'
                    remote.host = 'ec2-54-169-138-72.ap-southeast-1.compute.amazonaws.com'
                    remote.name = 'ip-172-31-28-109'
                    remote.identityFile = '~/ams/devops-up.pem'
                    remote.allowAnyHosts = 'true'
                    sshCommand remote: remote, command: 'aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 905140238863.dkr.ecr.ap-southeast-1.amazonaws.com'
                    sshCommand remote: remote, command: 'docker ps -f name=ams-docker -q | xargs --no-run-if-empty docker container stop'
                    sshCommand remote: remote, command: 'docker container ls -a -fname=ams-docker -q | xargs -r docker container rm'
                    sshCommand remote: remote, command: 'docker pull 905140238863.dkr.ecr.ap-southeast-1.amazonaws.com/devops2-test:latest'
//                     sshCommand remote: remote, command: 'docker run -d -p 80:80 -e APP_KEY="base64:3ilviXqB9u6DX1NRcyWGJ+sjySF+H18CPDGb3+IVwMQ=" --rm --name ams-docker 905140238863.dkr.ecr.ap-southeast-1.amazonaws.com/devops2-test:latest'
                    sshCommand remote: remote, command: 'docker stack deploy --compose-file /home/docker-compose.yml ams-docker'
                    sshCommand remote: remote, command: 'docker stack services ams-docker'
                    sshCommand remote: remote, command: 'curl http://localhost:80'
                    
                }
            } 
        }
 
        stage('Send Slack Message') {
            steps{
                script{
                    slackSend channel: 'devops-final', message: 'Test build from jenkinsfile'
                }
            }
        }
    }
}
