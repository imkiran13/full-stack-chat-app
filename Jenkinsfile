pipeline {
    agent any

    environment {
        SONAR_HOME = tool "sonarqube"
        KUBECONFIG = "$WORKSPACE/.kube/config"
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/imkiran13/full-stack-chat-app.git'
            }
        }

        stage('Code Quality Analysis') {
            steps {
                withSonarQubeEnv("sonarqube") {
                    sh "${SONAR_HOME}/bin/sonar-scanner -Dsonar.projectKey=fullstack-app -Dsonar.projectName=FullstackApp"
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    sh 'docker build -t imkiran13/chatapp-fronend ./frontend'
                    sh 'docker build -t imkiran13/chatapp-backend ./backend'
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh 'docker tag chatapp-fronend imkiran13/chatapp-fronend:latest'
                        sh 'docker tag chatapp-backend imkiran13/chatapp-backend:latest'
                        sh 'docker push imkiran13/chatapp-fronend:latest'
                        sh 'docker push imkiran13/chatapp-backend:latest'
                    }
                }
            }
        }

        stage('Set Up Minikube') {
            steps {
                script {
                    sh 'minikube start --driver=docker'
                    sh 'kubectl config use-context minikube'
                    sh 'kubectl create namespace fullstack-app || true'
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                script {
                    sh 'kubectl apply -n fullstack-app -f .'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                script {
                    sh 'kubectl get all -n fullstack-app'
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up Minikube...'
            sh 'minikube delete'
        }
    }
}
