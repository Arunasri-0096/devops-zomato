pipeline {
    agent any

    tools {
        nodejs 'node16.19'
    }

    environment {
        TRIVY_REPORT = "trivy-report.txt"
    }

    stages {

        stage("Clean Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Git Checkout") {
            steps {
                git url: 'https://github.com/Arunasri-0096/devops-zomato.git', branch: 'master'
            }
        }

        stage("Check Tool") {
            steps {
                sh 'ls -l'
                sh 'which dependency-check || echo "Not Found"'
            }
        }

        stage("Install Dependencies") {
            steps {
                sh 'npm install'
            }
        }

        stage("Build App") {
            steps {
                sh 'npm run build || true'
            }
        }

        stage("OWASP Dependency Check") {
            when {
                expression { return false }
            }
            steps {
                echo "Skipping OWASP"
            }
        }

        // ✅ SKIPPED TRIVY STAGE
        stage("Trivy File Scan") {
            when {
                expression { return false }
            }
            steps {
                echo "Skipping Trivy Scan"
            }
        }

        stage("Docker Build") {
            steps {
                sh 'docker build -t kastrov/zomato:latest .'
            }
        }

        stage("Push Docker Image") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker') {
                        sh 'docker push kastrov/zomato:latest'
                    }
                }
            }
        }

        stage("Deploy Container") {
            steps {
                sh '''
                docker rm -f zomato || true
                docker run -d --name zomato -p 3000:3000 kastrov/zomato:latest
                '''
            }
        }
    }

    post {
        always {
            emailext attachLog: true,
                subject: "'${currentBuild.result}'",
                body: """
                <html>
                <body>
                    <h2>Build Report</h2>
                    <p><b>Project:</b> ${env.JOB_NAME}</p>
                    <p><b>Build Number:</b> ${env.BUILD_NUMBER}</p>
                    <p><b>URL:</b> ${env.BUILD_URL}</p>
                </body>
                </html>
                """,
                to: 'srividyapsn2014@gmail.com',
                mimeType: 'text/html',
                attachmentsPattern: "${TRIVY_REPORT}"
        }

        success {
            echo "✅ Pipeline Success"
        }

        failure {
            echo "❌ Pipeline Failed"
        }
    }
}
