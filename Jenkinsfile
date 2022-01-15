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
        stage('Docker build') {
            steps {
                script {
                  dockerImage = docker.build registry
                }
            }
        }
        stage('Docker push') {
            steps {
                script {
                  sh 'aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 905140238863.dkr.ecr.ap-southeast-1.amazonaws.com'
                  sh 'docker push 905140238863.dkr.ecr.ap-southeast-1.amazonaws.com/devops2-test:latest'
                }
            }
        }
        // Stopping Docker containers for cleaner Docker run
        stage('stop previous containers') {
             steps {
                sh 'docker ps -f name=snipeit -q | xargs --no-run-if-empty docker container stop'
                sh 'docker container ls -a -fname=snipeit -q | xargs -r docker container rm'
             }
        }
        stage('Docker Run') {
            steps{
                script {
                    sh 'docker run -d -p 8096:80 --rm --name snipeit 905140238863.dkr.ecr.ap-southeast-1.amazonaws.com/devops2-test:latest'
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

        stage('Remote web server'){
            steps{
                script{
                    sh 'bash ~/ams/start.sh'
                }
            }
        }
    }
}
