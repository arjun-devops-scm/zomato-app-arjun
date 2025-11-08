pipeline {
    agent { label 'jenkins-agent' }

    tools {
        nodejs 'nodejs-23'
    }

    stages {
        stage('Installing dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm run test'
            }
        }

        stage('Trivy File System Scan') {
            steps {
                sh 'trivy fs --format json -o trivy-files-scan-report.json .'
                archiveArtifacts artifacts: 'trivy-files-scan-report.json', fingerprint: true
            }
        }

        stage('Sonar Analysis') {
            steps {
                script {
                    def SONAR_SCANNER_HOME = tool name: 'sonar-scanner'
                    withSonarQubeEnv('sonar') {
                        sh "${SONAR_SCANNER_HOME}/bin/sonar-scanner -Dsonar.language=js -Dsonar.projectKey=zomato"
                    }
                }
            }
        }

        stage('Sonar Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t arjundocker92/zomato:${BUILD_NUMBER} ."
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format json -o zomato-image-trivy-scan-report.json arjundocker92/zomato:${BUILD_NUMBER}"
                archiveArtifacts artifacts: 'zomato-image-trivy-scan-report.json', fingerprint: true
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            docker push arjundocker92/zomato:${BUILD_NUMBER}
                        """
                    }
                }
            }
        }

        stage('Deploy Zomato App') {
            steps {
                script {
                    def SERVER_IP = "65.0.68.100"
                    sshagent (credentials: ['deploy-sever-creds']) {
                        sh """
                            ssh -o StrictHostKeyChecking=no root@${SERVER_IP} '
                                echo "Pulling latest image...";
                                docker pull arjundocker92/zomato:${BUILD_NUMBER};
                                docker stop zomato || true;
                                docker rm zomato || true;
                                docker run -d --name zomato -p 3000:3000 arjundocker92/zomato:${BUILD_NUMBER};
                            '
                        """
                    }
                }
            }
        }
    }
}
