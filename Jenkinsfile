pipeline {
    agent {
        node {
            label 'Slave'
        }
    }
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    environment {
        REGISTRY = 'harbor.y4test.local'
        PROJECT_NAME = 'register-app'
        APP_NAME = 'register-app-pipeline'
        RELEASE = '1.0.2'
        DOCKER_PASS = 'jenkins-harbor-user'
        IMAGE_NAME = "${REGISTRY}" + '/' + "${PROJECT_NAME}" + '/' + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from SCM') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/hkarademir/register-app'
            }
        }
        stage('Build Application') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test Application') {
            steps {
                sh 'mvn test'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }
        stage('Build & Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("https://${REGISTRY}", "${DOCKER_PASS}") {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                    docker.withRegistry("https://${REGISTRY}", "${DOCKER_PASS}") {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }
        stage('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}
