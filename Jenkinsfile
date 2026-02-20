pipeline {
    agent any

    stages {
        stage('Initialize') {
            steps {
                // Workspace cleanup at the start
                cleanWs()
                echo "Workspace cleaned. Starting fresh build..."
            }
        }

        stage('Security Analysis') {
            parallel {
                stage('SCA') {
                    steps {
                        withCredentials([string(credentialsId: 'nvdApiKey', variable: 'nvdApiKey')]) {                        
                        echo "Running SCA (OWASP Dependency-Check)..."
                  //      sh 'npm install bcrypt@latest'
                        sh 'npm install'
                        sh 'dc --nvdApiKey ${nvdApiKey} --project vulnNode -s . -f ALL -o ../output'
                        echo "SCA Done."
                    }
                }
                stage('SAST') {
                    steps {
                        echo "Running SAST (Snyk, Bearer)..."
                        sh 'bearer scan . -f json --output ../output/bearer/vulnNode.json'
                        echo "SAST Done"
                    }
                }
            }
        }

        stage('Docker Image Build & Scan') {
            steps {
                echo "Building Docker image..."
                sh 'docker compose build'
                
                echo "Scanning Docker Image for vulnerabilities (e.g., Trivy or Grype)..."
                sh 'trivy image vulnerable-node-vulnerable_node -o ../output/trivy/vulnNode.json -f json'
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
                echo "Deploying and running Dynamic Application Security Testing (OWASP ZAP)..."
                sh 'docker run --rm -v $(pwd):/zap/wrk/:rw -t zaproxy/zap-stable:latest zap-baseline.py -t http://localhost:3000 -r scan-report.html'
            }
        }
    }
}
