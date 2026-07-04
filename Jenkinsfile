pipeline {
    agent { label 'k8s-agent' }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Running on agent: ${env.NODE_NAME}"
            }
        }

        stage('Secret Text Demo') {
            steps {
                withCredentials([string(
                    credentialsId: 'my-api-token',
                    variable: 'API_TOKEN'
                )]) {
                    echo "✅ Secret Text credential loaded successfully"
                    sh 'echo The token value is: $API_TOKEN'
                    sh 'echo Length of token: ${#API_TOKEN}'
                }
            }
        }

        stage('Username Password Demo') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    echo "✅ Username/Password credential loaded"
                    sh 'echo Username is: $DOCKER_USER'
                    sh 'echo Password is: $DOCKER_PASS'
                    sh 'echo Both above lines will show **** for password'
                }
            }
        }

        stage('Credentials Outside Block') {
            steps {
                echo "Trying to access credential outside withCredentials block..."
                sh 'echo API_TOKEN value outside block: ${API_TOKEN:-NOT_SET}'
                sh 'echo This proves credentials only exist INSIDE the block'
            }
        }

        stage('Multiple Credentials Together') {
            steps {
                withCredentials([
                    string(credentialsId: 'my-api-token', variable: 'API_TOKEN'),
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    echo "✅ Both credentials loaded in same block"
                    sh 'echo Using API token and Docker creds simultaneously'
                    sh 'echo Docker user: $DOCKER_USER'
                    sh 'echo API Token masked: $API_TOKEN'
                }
            }
        }

    }

    post {
        success {
            echo "✅ All credential stages passed! Secrets were masked correctly."
        }
        failure {
            echo "❌ Check which stage failed - likely credential ID mismatch"
        }
        always {
            deleteDir()
        }
    }
}
