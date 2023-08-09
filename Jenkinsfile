pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "nitejain/train-schedule"
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            when {
                branch 'master'
            }
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                withKubeConfig([credentialsId: 'jenkins-deploy', serverUrl: 'https://172.31.5.104:6443']) {
                sh 'kubectl apply -f train-schedule-kube-canary.yml -n jenkins-deploy'
                }
            }
        }
        stage('DeployToProduction') {
            steps {
                withKubeConfig([credentialsId: 'jenkins-deploy', serverUrl: 'https://172.31.5.104:6443']) {
                sh 'kubectl apply -f train-schedule-kube-canary.yml -n jenkins-deploy'
                }
            }
        }
    }
}
