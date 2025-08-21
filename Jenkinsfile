pipeline {
    agent any

    tools {
        nodejs 'node16'   // Ensure NodeJS 16 is configured in Jenkins Global Tools
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/muhammadaliazhar/fullstack-bank.git'
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                withCredentials([string(credentialsId: 'NVD_API_KEY', variable: 'NVD_KEY')]) {
                    dependencyCheck additionalArguments: "--scan ./app --exclude **/node_modules/** --nvdApiKey=${NVD_KEY} --data /var/lib/jenkins/odc-data --disableYarnAudit", odcInstallation: 'DC'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                }
            }
        }

        stage('Trivy FS Scanning') {
            steps {
                sh "trivy fs ."
            }
        }

        stage('Install Dependencies') {
            parallel {
                stage('Backend') {
                    steps {
                        dir('app/backend') {
                            sh "npm ci"
                        }
                    }
                }
                stage('Frontend') {
                    steps {
                        dir('app/frontend') {
                            sh "npm ci"
                        }
                    }
                }
            }
        }

        stage('Frontend Build') {
            steps {
                dir('app/frontend') {
                    sh "npm run build"
                }
            }
        }

        stage('SonarQube Scanning') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectName=Bank \
                        -Dsonar.projectKey=Bank \
                        -Dsonar.sources=.
                    """
                }
            }
        }

        stage('Deploy to Container') {
            steps {
                dir('app'){
                    sh "docker compose up -d"
                }
            }
        }
    }
}
