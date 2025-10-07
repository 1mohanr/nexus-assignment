pipeline {
    agent any

    environment {
        APP_NAME = "my-app"
        SONARQUBE_SERVER = "SonarQube" // Name of your SonarQube installation in Jenkins
        SONAR_TOKEN_ID = "sonarqube-token" // Jenkins credential ID for SonarQube token
        NEXUS_CREDENTIALS_ID = "nexus-credentials"
        NEXUS_URL = "http://54.206.135.18:30080/"
        DOCKER_IMAGE = "381492179072.dkr.ecr.ap-southeast-2.amazonaws.com/${APP_NAME}:latest"
        AWS_REGION = "ap-southeast-2"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/1mohanr/nexus-assignment.git'
            }
        }

        stage('Code Scan') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh """
                    ${tool 'SonarQubeScanner'}/bin/sonar-scanner \
                        -Dsonar.projectKey=${APP_NAME} \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://3.106.241.242:30466/ \
                        -Dsonar.login=\$(aws secretsmanager get-secret-value --secret-id ${SONAR_TOKEN_ID} --query SecretString --output text)
                    """
                }
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Publish to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUS_URL.replace('http://','')}",
                    groupId: 'com.example',
                    version: '1.0.0',
                    repository: 'maven-releases',
                    credentialsId: "${NEXUS_CREDENTIALS_ID}",
                    artifacts: [
                        [artifactId: "${APP_NAME}", type: 'zip', file: "dist/${APP_NAME}.zip"]
                    ]
                )
            }
        }

        stage('Docker Build & Push to ECR') {
            steps {
                script {
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin 381492179072.dkr.ecr.${AWS_REGION}.amazonaws.com"
                    sh "docker build -t ${APP_NAME} ."
                    sh "docker tag ${APP_NAME}:latest ${DOCKER_IMAGE}"
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
        failure {
            echo "Pipeline failed."
        }
        success {
            echo "Pipeline succeeded."
        }
    }
}

