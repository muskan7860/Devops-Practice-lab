pipeline {
    agent any
    
    stages {
        stage('Hello'){
            steps {
                echo 'stage 1: Hello from Feature Branch!'
                
            }
        }
        stage('Who Am I') {
            steps {
                  sh 'whoami'
                  sh 'pwd'
                  sh 'ls -la'
            }
          
        }
        stage('Environment Check') {
            steps {
                echo "Build Number: ${env.BUILD_NUMBER}"
                echo "Job Name: ${env.JOB_NAME}"
                echo "Workspace: ${env.WORKSPACE}"
                echo "Git Branch: ${env.GIT_BRANCH}"
            }
        }
    }
    post {
        success {
            echo 'All stages passed! Great job'
        }
        failure {
            echo 'something went wrong. check the stage that turned red.'
        }
        }
    }
    

