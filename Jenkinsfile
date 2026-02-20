pipeline {
    agent any

    stages {

        stage('Initialize') {
            steps {
                sh 'mkdir output'
            }
        }

        stage('Security Analysis') {
            parallel {

                stage('SCA') {
                    steps {
                        withCredentials([string(credentialsId: 'nvdApiKey', variable: 'nvdApiKey')]) {
                            echo "Running SCA (OWASP Dependency-Check)..."
                            sh 'npm install'
                            sh "dc --nvdApiKey ${nvdApiKey} --project vulnNode -s . -f ALL -o output/"
                            echo "SCA Done."
                        }
                    }
                }

                stage('SAST') {
                    steps {
                        echo "Running SAST (Snyk, Bearer)..."
                        sh 'bearer scan . -f json --output output/vulnNode.json'
                        echo "SAST Done"
                    }
                }
            }
        }

        stage('Docker Image Build & Scan') {
            steps {
                echo "Building Docker image..."
                sh 'docker compose build'

                echo "Scanning Docker Image..."
                sh 'trivy image vulnerable-node-vulnerable_node -o output/vulnNode.json -f json'
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying Application..."
                sh 'docker compose up -d'
            }
        }

        stage('DAST') {
            steps {
                echo "Running OWASP ZAP DAST..."
                sh 'docker run --rm -v $(pwd):/zap/wrk/:rw -t zaproxy/zap-stable:latest zap-baseline.py -t http://localhost:3000 -r scan-report.html'
            }
        }
    }
}
