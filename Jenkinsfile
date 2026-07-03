pipeline {
    agent none

    stages {

        stage('Stage on Master') {
            agent { label 'built-in' }
            steps {
                echo "This stage runs on: ${env.NODE_NAME}"
                sh 'hostname'
                echo "Master workspace: ${env.WORKSPACE}"
            }
        }

        stage('Stage on k8s-agent') {
            agent { label 'k8s-agent' }
            steps {
                echo "This stage runs on: ${env.NODE_NAME}"
                sh 'hostname'
                echo "Agent workspace: ${env.WORKSPACE}"
                sh 'echo I am running inside a Kubernetes Pod!'
            }
        }

        stage('Back to k8s-agent') {
            agent { label 'k8s-agent' }
            steps {
                echo "Still on: ${env.NODE_NAME}"
                sh 'echo Same agent, different stage'
            }
        }

    }

    post {
        always {
            echo "Pipeline finished. Stages ran on different agents!"
        }
    }
}
