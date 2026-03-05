pipeline {
    agent any

    stages {
           stage('Git Checkout') {
            steps {
                git 'https://github.com/rohitnayal/nodejs.git'
            }
        }

     stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {  
                    sh "${tool 'SonarQube Scanner'}/bin/sonar-scanner -Dsonar.projectKey=demo-app -Dsonar.sources=."
                }
            }
        }
    

        stage('Build Docker Image') {
            steps {
                sh "docker build -t demo-app:${BUILD_NUMBER} -f Dockerfile src/"
            }
        }
   
        stage('Push to Nexus') {
            steps {
                script {
                    docker.withRegistry('http://localhost:8083', 'nexus-credentials') {
                        sh "docker tag demo-app:${BUILD_NUMBER} localhost:8083/demo-app:${BUILD_NUMBER}"
                        sh "docker push localhost:8083/demo-app:${BUILD_NUMBER}"
                    }
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                script {
                    def imageTag = "localhost:8083/demo-app:${BUILD_NUMBER}"
                    sh "docker tag demo-app:${BUILD_NUMBER} demo-app:latest"
                    sh "kubectl apply -f deployment.yaml"
                    sh "kubectl apply -f service.yaml"
                    sh "kubectl rollout status deployment/demo-app"
                }
            }
        }

    } 
} 