pipeline {
    agent any

    tools {
        jdk 'jdk21'
        maven 'MAVEN'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: "https://github.com/gangassault343/superlab-private.git",
                    credentialsId: "github-access",
                    branch: "main"
            }
        }

        stage('Maven Build') {
            steps {
                echo 'Building project'
                sh "mvn clean verify -Dtest='!FormUITesttt'"
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Please check the logs."
        }
    }
}