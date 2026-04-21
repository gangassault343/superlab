pipeline {
    agent any
    environment {        
        SONAR_SCANNER = tool 'SONAR'
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
                echo 'Building project1'
                sh "mvn clean verify -Dtest='!FormUITest'"
            }
        }
        stage('SonarCloud Scan') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    echo '🔍 Running SonarCloud analysis (coverage fully disabled)...'
                    sh """
                        ${SONAR_SCANNER}/bin/sonar-scanner \
                          -Dsonar.projectKey=gangassaultsonar_sonar \
                          -Dsonar.organization=gangassaultsonar \
                          -Dsonar.sources=src/main/java \
                          -Dsonar.tests=src/test/java \
                          -Dsonar.java.binaries=target/classes \
                          -Dsonar.coverage.exclusions=**/*.java \
                          -Dsonar.coverage.newCode.requiredCoverage=0 \
                          -Dsonar.newCode.period=1 \
                          -Dsonar.qualitygate.wait=false \
                          -Dsonar.host.url=https://sonarcloud.io
                    """
                }
            }
        }
        stage('Check Sonar Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "❌ Quality Gate failed: ${qg.status}"
                        }
                    }
                }
            }
        }
        stage('Docker Build and Run') {
            steps {
                echo '🐳 Building and running Docker container...'
                sh """
                    docker build -t superlab:${BUILD_NUMBER} .
                    docker ps -q --filter "publish=8081" | grep -q . && docker rm -f \$(docker ps -q --filter "publish=8081") || echo "No container on port 8081"
                    docker run -d -p 8081:8080 --name superlab-app-${BUILD_NUMBER} superlab:${BUILD_NUMBER}
                    sleep 10
                """
            }
        }
        stage('Selenium Headless GUI Test') {
            steps {
                echo '🚀 Running Selenium GUI tests...'
                sh "${MAVEN_HOME}/bin/mvn -Dtest=FormUITest test -DfailIfNoTests=false"
            }
        }
        stage('Push Docker Image to Docker Hub') {
            steps {
                echo '📦 Pushing image to Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag superlab:${BUILD_NUMBER} gangassault343/superlab:${BUILD_NUMBER}
                        docker push gangassault343/superlab:${BUILD_NUMBER}
                    """
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