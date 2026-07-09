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

      stage('Build Image') {
        steps {
            // Use double quotes and the dynamic variable
            sh "docker build -t devops-cicd-lab:${IMAGE_TAG} ."
        }
    }

    stage('Push Image') {
        steps {
            withDockerRegistry([credentialsId: 'docker-hub-credentials', url: 'https://index.docker.io/v1/']) {
                // Update to use variables instead of hardcoded '20'
                sh "docker tag devops-cicd-lab:${IMAGE_TAG} mcakash/devops-cicd-lab:${IMAGE_TAG}"
                sh "docker push mcakash/devops-cicd-lab:${IMAGE_TAG}"
            }
        }
    }

    stage('Deploy Container') {
        steps {
            sh """
            docker rm -f ${CONTAINER_NAME} || true
            docker run -d --name ${CONTAINER_NAME} -p 8082:8080 mcakash/devops-cicd-lab:${IMAGE_TAG}
            """
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

