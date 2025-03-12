pipeline {
    agent any

    environment {
        branch_name = 'main'
        repo_url = 'https://github.com/snaatak-Zero-Downtime-Crew/salary-api.git'
        gitleaks_report_name = "gitleaks-report.json"
        recipient_email = 'jnikita647@gmail.com'
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Git Clone') {
            steps {
                git branch: branch_name, url: repo_url, credentialsId: 'snaatak'
            }
        }

        stage('Download and Install Gitleaks') {
            steps {
                sh '''
                    wget https://github.com/gitleaks/gitleaks/releases/download/v8.24.0/gitleaks_8.24.0_linux_x64.tar.gz
                    tar -xzvf gitleaks_8.24.0_linux_x64.tar.gz
                    chmod +x gitleaks
                    sudo mv gitleaks /usr/local/bin/
                '''
            }
        }

        stage('Gitleaks Scan') {
            steps {
                sh 'gitleaks detect -r ${gitleaks_report_name}'
            }
        }

        stage('Publish Gitleaks Report') {
            steps {
                archiveArtifacts artifacts: gitleaks_report_name, fingerprint: true
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
            archiveArtifacts artifacts: gitleaks_report_name, allowEmptyArchive: true
        }

        success {
            emailext(
                to: "${recipient_email}",
                subject: "Jenkins Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """Hello,

The Jenkins pipeline **${env.JOB_NAME}** has completed successfully on **Build #${env.BUILD_NUMBER}**.

**Build Details:**
- **Job Name:** ${env.JOB_NAME}
- **Build Number:** ${env.BUILD_NUMBER}
- **Build URL:** ${env.BUILD_URL}

You can find the Gitleaks scan report attached.

Best regards,
Jenkins CI
""",
                attachmentsPattern: gitleaks_report_name
            )
        }

        failure {
            emailext(
                to: "${recipient_email}",
                subject: "Jenkins Build FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """Hello,

The Jenkins pipeline **${env.JOB_NAME}** has failed on **Build #${env.BUILD_NUMBER}**.

**Job Details:**
- **Job Name:** ${env.JOB_NAME}
- **Build Number:** ${env.BUILD_NUMBER}
- **Build URL:** ${env.BUILD_URL}

Please review the attached Gitleaks scan report for more details.

Regards,
Jenkins CI
""",
                attachmentsPattern: gitleaks_report_name
            )
        }
    }
}
