pipeline {
    agent any

    environment {
        APP_NAME = "my-app"
        DOCKER_IMAGE = "381492179072.dkr.ecr.ap-southeast-2.amazonaws.com/${APP_NAME}:${env.BUILD_NUMBER}"
        SONARQUBE_SERVER = "SonarQubeServer"  // Jenkins SonarQube server ID
        NEXUS_URL = "http://54.206.135.18:30080/repository/maven-releases/"
    }

    triggers {
        githubPush()
    }

    stages {

        // Stage 1: Checkout Code
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/1mohanr/nexus-assignment.git'
            }
        }

        // Stage 2: Code Scan (SonarQube)
        stage('Code Scan') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh "sonar-scanner -Dsonar.projectKey=${APP_NAME} -Dsonar.sources=. -Dsonar.login=sonarqube-token -Dsonar.host.url=http://54.206.135.18:30080/"
                }
            }
        }

        // Stage 3: Build (Node.js)
        stage('Build') {
            steps {
                sh "npm install"
                sh "npm test"
            }
        }

        // Stage 4: Store Artifacts in Nexus
        stage('Publish to Nexus') {
            steps {
                sh """
                mkdir -p dist
                zip -r dist/${APP_NAME}.zip *
                """
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '54.206.135.18:30080',
                    repository: 'maven-releases',
                    credentialsId: 'nexus-credentials',
                    groupId: 'com.example',
                    artifactId: "${APP_NAME}",
                    version: "${env.BUILD_NUMBER}",
                    type: 'zip',
                    file: "dist/${APP_NAME}.zip"
                )
            }
        }

        // Stage 5: Docker Image Build & Push to AWS ECR
        stage('Docker Build & Push to ECR') {
            steps {
                script {
                    // Login to AWS ECR
                    sh "aws ecr get-login-password --region ap-southeast-2 | docker login --username AWS --password-stdin 381492179072.dkr.ecr.ap-southeast-2.amazonaws.com"

                    // Build Docker image
                    sh "docker build -t ${APP_NAME}:${env.BUILD_NUMBER} ."

                    // Tag Docker image for ECR
                    sh "docker tag ${APP_NAME}:${env.BUILD_NUMBER} ${DOCKER_IMAGE}"

                    // Push Docker image to ECR
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed."
        }
    }
}

