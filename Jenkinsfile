pipeline {
    agent any

    environment {
        SONAR_SCANNER = tool 'SonarScanner'
    }

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
                sh "mvn clean verify -Dtest='!FormUITest'"
            }
        }
        stage('SonarQube Code Quality Scan') {
            steps {
                withSonarQubeEnv('SONAR') {
                echo '🔍 Running SonarCloud analysis (coverage disabled for now)...'
                sh """
                    ${SONAR_SCANNER}/bin/sonar-scanner \
                        -Dsonar.projectKey=sonar superlab_sonarqubeproject \
                        -Dsonar.organization=sonar superlab \
                        -Dsonar.sources=src/main/java \
                        -Dsonar.tests=src/test /java \
                        -Dsonar.java.binaries=target/classes \
                        -Dsonar.coverage.exclusions=**/*.java \
                        -Dsonar.coverage.newCode.requiredCoverage=0 \
                        -Dsonar.newCode.period=1 \
                        -Dsonar.qualitygate.wait=true \
                        -Dsonar.host.url=https: //sonarcloud.io
                    """
                }
            }
        }
        stage('Check Sonar Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate( )
                        if (qg.status != 'OK') {
                            error "❌ Quality Gate failed: ${qg.status}"
                        }
                    }
                }
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