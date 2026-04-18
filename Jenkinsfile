pipeline {
    agent any

   environment {
        MAVEN_HOME = tool 'MAVEN'
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "Cloning Private Github repo"
                git url: "https://github.com/gangassault343/superlab-private.git", credentialsId: "github-access", branch: "main"
            }
        }

        stage('Maven Build') {
            steps {
                echo 'Building project'
                sh "${MAVEN_HOME}/bin/mvn clean verify -Dtest=!FormUITest"
            }
        }

        // ➕ Add more stages as needed
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