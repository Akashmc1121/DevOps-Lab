pipeline {
    agent any

    tools {
        Maven 'Maven3'   
    }

    environment {
        IMAGE_NAME = "devops-cicd-lab"
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
        CONTAINER_NAME = "devops-cicd-lab-container"
        DOCKERHUB_REPO  = "mcakash/devops-cicd-lab" 
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Akashmc1121/DevOps-Lab.git'
                
            }
        }

        stage('Build with Maven') {
            steps {
                // FIXED: Explicitly target the subfolder pom config
                sh 'mvn -f devops-cicd-lab/pom.xml clean compile'
            }
        }

        stage('Run Unit Tests') {
            steps {
                // FIXED: Explicitly target the subfolder pom config
                sh 'mvn -f devops-cicd-lab/pom.xml test'
            }
            post {
                always {
                    // FIXED: Target the subfolder target outputs
                    junit 'devops-cicd-lab/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package JAR') {
            steps {
                // FIXED: Explicitly target the subfolder pom config
                sh 'mvn -f devops-cicd-lab/pom.xml package -DskipTests'
                archiveArtifacts artifacts: 'devops-cicd-lab/target/*.jar', fingerprint: true
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // FIXED: Tell the docker builder plugin to target the subfolder path root
                    dockerImage = docker.build("${DOCKERHUB_REPO}:${IMAGE_TAG}", "devops-cicd-lab")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://docker.io', 'dockerhub-creds') {
                        dockerImage.push("${IMAGE_TAG}")
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage('Deploy Container') {
            steps {
                sh """
                    docker rm -f ${CONTAINER_NAME} || true
                    docker run -d --name ${CONTAINER_NAME} -p 8082:8080 ${DOCKERHUB_REPO}:${IMAGE_TAG}
                """
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully. Container is running.'
        }
        failure {
            echo 'Pipeline failed. Check the stage logs above.'
        }
    }
}
