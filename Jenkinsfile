pipeline {
    agent { label 'k8s-agent' }

    stages {

        stage('Prove We Are on k8s-agent') {
            steps {
                echo "============================================"
                echo "Agent Name: ${env.NODE_NAME}"
                echo "Workspace:  ${env.WORKSPACE}"
                echo "Build No:   ${env.BUILD_NUMBER}"
                echo "============================================"
                sh 'whoami'
                sh 'hostname'
                sh 'echo My IP is: $(hostname -i)'
            }
        }

        stage('Check Kubernetes Identity') {
            steps {
                echo "Checking if we are inside a Kubernetes pod..."
                sh 'cat /etc/hostname'
                sh 'echo Namespace: $(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace)'
                sh 'ls /var/run/secrets/kubernetes.io/serviceaccount/'
            }
        }

        stage('Check Available Tools') {
            steps {
                echo "What tools does this agent have?"
                sh 'java -version'
                sh 'git --version'
                sh 'curl --version | head -1'
                sh 'which wget || echo wget not installed'
                sh 'which docker || echo docker not installed'
                sh 'which kubectl || echo kubectl not installed'
            }
        }

        stage('Create File in Workspace') {
            steps {
                echo "Creating a file in the agent workspace..."
                sh '''
                    echo "This file was created by Jenkins build ${BUILD_NUMBER}" > myfile.txt
                    echo "Agent: ${NODE_NAME}" >> myfile.txt
                    echo "Date: $(date)" >> myfile.txt
                    cat myfile.txt
                '''
                echo "File created successfully inside the k8s pod workspace!"
            }
        }

        stage('Show Workspace Contents') {
            steps {
                sh 'ls -la'
                sh 'pwd'
                sh 'df -h | head -5'
            }
        }

    }

    post {
        success {
            echo "Pipeline ran successfully on agent: ${env.NODE_NAME}"
        }
        failure {
            echo "Pipeline failed on agent: ${env.NODE_NAME}"
        }
        always {
            echo "Cleaning up workspace..."
            cleanWs()
        }
    }
}
