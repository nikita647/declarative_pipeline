node {
    def branch_name = 'main'
    def repo_url = 'https://github.com/snaatak-Zero-Downtime-Crew/salary-api.git'
    def gitleaks_report_name = "gitleaks-report.json"
    def recipient_email = 'jnikita647@gmail.com'
    
    try {
        stage('Clean Workspace') {
            deleteDir()
        }
        
        stage('Git Clone') {
            checkout([$class: 'GitSCM', 
                      branches: [[name: branch_name]], 
                      userRemoteConfigs: [[url: repo_url, credentialsId: 'snaatak']]
            ])
        }
        
        stage('Download and Install Gitleaks') {
            sh '''
                wget https://github.com/gitleaks/gitleaks/releases/download/v8.24.0/gitleaks_8.24.0_linux_x64.tar.gz
                tar -xzvf gitleaks_8.24.0_linux_x64.tar.gz
                chmod +x gitleaks
                sudo mv gitleaks /usr/local/bin/
            '''
        }
        
        stage('Gitleaks Scan') {
            sh "gitleaks detect -r ${gitleaks_report_name}"
        }
        
        stage('Publish Gitleaks Report') {
            archiveArtifacts artifacts: gitleaks_report_name, fingerprint: true
        }
        
        stage('Send Success Email') {
            emailext(
                to: recipient_email,
                subject: "Jenkins Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """Hello,

The Jenkins pipeline *${env.JOB_NAME}* has completed successfully on *Build #${env.BUILD_NUMBER}*.

*Build Details:*
- *Job Name:* ${env.JOB_NAME}
- *Build Number:* ${env.BUILD_NUMBER}
- *Build URL:* ${env.BUILD_URL}

You can find the Gitleaks scan report attached.

Best regards,
Jenkins CI
""",
                attachmentsPattern: gitleaks_report_name
            )
        }
    } catch (Exception err) {
        stage('Send Failure Email') {
            emailext(
                to: recipient_email,
                subject: "Jenkins Build FAILURE: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """Hello,

The Jenkins pipeline *${env.JOB_NAME}* has failed on *Build #${env.BUILD_NUMBER}*.

*Job Details:*
- *Job Name:* ${env.JOB_NAME}
- *Build Number:* ${env.BUILD_NUMBER}
- *Build URL:* ${env.BUILD_URL}

Please review the attached Gitleaks scan report for more details.

Regards,
Jenkins CI
""",
                attachmentsPattern: gitleaks_report_name
            )
        }
        throw err
    }
}
