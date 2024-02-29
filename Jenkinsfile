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
        HARBOR_NAMESPACE = 'ks-devops-harbor'
        APP_NAME = 'register-app-pipeline'
        RELEASE = '1.0.0'
        HARBOR_CREDENTIALS = credentials('jenkins-harbor-user')
        IMAGE_NAME = "${REGISTRY}" + '/' + "${HARBOR_NAMESPACE}" + '/' + "${APP_NAME}"
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
        stage('Build') {
            steps {
                sh 'docker build -t $REGISTRY/$HARBOR_NAMESPACE/$APP_NAME:$RELEASE .'
            }
        }
        stage('Docker Login') {
            steps {
                sh 'echo $HARBOR_CREDENTIALS_PSW | docker login $REGISTRY -u $HARBOR_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Push') {
            steps {
                sh 'docker push $REGISTRY/$HARBOR_NAMESPCAE/$APP_NAME:$RELEASE'
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}
