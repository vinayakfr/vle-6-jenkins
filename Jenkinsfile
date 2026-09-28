pipeline {
    agent any

    tools {
        maven 'Maven'   // name configured in Manage Jenkins → Tools
    }

    triggers {
        pollSCM('H/2 * * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Run') {
            steps {
                sh 'java -jar target/jenkins-lab-1.0.jar'
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
    }
}
