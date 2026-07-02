pipeline {
    agent any

    environment {
        APP_NAME  = 'my-practice-app'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Code checked out successfully"
            }
        }

        stage('Build') {
            steps {
                echo "Building application: ${APP_NAME}"
                echo "Simulating: mvn clean package -DskipTests"
                sh 'echo BUILD SUCCESS'
            }
        }

        stage('Code Analysis') {
            steps {
                echo "Running SonarQube code analysis"
                echo "Simulating: mvn sonar:sonar"
                sh 'echo SONARQUBE SCAN COMPLETE - Quality Gate PASSED'
            }
        }

        stage('Docker Build') {
            steps {
                echo "Building Docker Image: ${APP_NAME}:${IMAGE_TAG}"
                echo "Simulating: docker build -t ${APP_NAME}:${IMAGE_TAG} ."
                sh 'echo DOCKER IMAGE BUILT SUCCESSFULLY'
            }
        }

        stage('Docker Push') {
            steps {
                echo "Pushing Docker Image to registry"
                echo "Simulating: docker push ${APP_NAME}:${IMAGE_TAG}"
                sh 'echo DOCKER IMAGE PUSHED SUCCESSFULLY'
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                echo "Deploying ${APP_NAME}:${IMAGE_TAG} to server..."
                sh 'echo Deployment complete'
            }
        }
    }

    post {
        success {
            echo "Pipeline Passed! App: ${APP_NAME}, Build: ${IMAGE_TAG}"
        }

        failure {
            echo "Pipeline Failed! Check the red stage above."
        }

        always {
            echo "Cleanup complete. Workspace will be cleared."
            cleanWs()
        }
    }
}
