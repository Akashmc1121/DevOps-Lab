pipeline {
    agent any

    tools {
        // Name must match the installation name in Manage Jenkins > Tools
        maven 'Maven3'   
    }

    environment {
        IMAGE_NAME = "devops-cicd-lab"
        IMAGE_TAG  = "${env.BUILD_NUMBER}"
        CONTAINER_NAME = "devops-cicd-lab-container"
        DOCKERHUB_REPO  = "mcakash/devops-cicd-lab" 
    }

    stages {
        // FIXED: Removed the manual redundant checkout stage to fix the 'fatal: not in a git directory' error

        stage('Build with Maven') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Package JAR') {
            steps {
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // FIXED: Uses the native Docker plugin API to build natively without shell lookups
                    dockerImage = docker.build("${DOCKERHUB_REPO}:${IMAGE_TAG}", ".")
                }
            }
        }

     stage('Push to Docker Hub') {
    steps {
        script {
            // This wrapper automatically handles 'docker login' and 'docker logout'
            withDockerRegistry([credentialsId: 'dockerhub-creds', url: '']) {
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKERHUB_REPO}:${IMAGE_TAG}"
                sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${DOCKERHUB_REPO}:latest"
                sh "docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}"
                sh "docker push ${DOCKERHUB_REPO}:latest"
            }
        }
    }
}
        stage('Deploy Container') {
            steps {
                // FIXED: Changed host port to 8082 to avoid port conflicts with your native services on 8080
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
        // FIXED: Removed the global 'always' shell step to prevent pipeline architecture crash
    }
}

