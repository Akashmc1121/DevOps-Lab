pipeline {
    agent any
    
    tools {
        // Name must match the installation name in Manage Jenkins > Tools
        maven 'Maven3'
    }
    
    environment {
        IMAGE_NAME     = "devops-cicd-lab"
        IMAGE_TAG      = "${env.BUILD_NUMBER}"
        CONTAINER_NAME = "devops-cicd-lab-container"
        DOCKERHUB_REPO = "mcakash/devops-cicd-lab"
    }
    
    stages {
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
        
        stage('Build Image') {
            steps {
                sh "docker build -t devops-cicd-lab:${IMAGE_TAG} ."
            }
        }
        
        stage('Push Image') {
            steps {
                withDockerRegistry([credentialsId: 'docker-hub-credentials', url: 'https://index.docker.io/v1/']) {
                    sh "docker tag devops-cicd-lab:${IMAGE_TAG} ${DOCKERHUB_REPO}:${IMAGE_TAG}"
                    sh "docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}"
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
    } // End of stages
    
    post {
        success {
            echo 'Pipeline completed successfully. Container is running.'
        }
        failure {
            echo 'Pipeline failed. Check the stage logs above.'
        }
    }
} // End of pipeline
