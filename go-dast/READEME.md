pipeline {
    agent any

    stages {
        stage('test') {
            steps {
                sh 'zap.sh -cmd -quickurl http://54.82.9.247:8080/employee/health -port 8090 -quickprogress -quickout /var/lib/jenkins/out2.html'
            }
        }
        
        stage('publish') {
            steps {
                publishHTML([
                    allowMissing: false, 
                    alwaysLinkToLastBuild: false, 
                    keepAll: false, 
                    reportDir: '/var/lib/jenkins/', 
                    reportFiles: 'out2.html', 
                    reportName: 'HTML Report', 
                    reportTitles: ''
                ])
            }
        }
    }
     post {
        always {
            echo 'Pipeline completed.'
            archiveArtifacts artifacts: 'target/dependency-check-report.html', allowEmptyArchive: true
        }

        success {
            emailext(
        attachmentsPattern: 'target/dependency-check-report.html',
        body: """Hello,

The Jenkins pipeline **${env.JOB_NAME}** has completed successfully on **Build #${env.BUILD_NUMBER}**.

**Build Details:**  
- **Job Name:** ${env.JOB_NAME}  
- **Build Number:** ${env.BUILD_NUMBER}  
- **Build URL:** ${env.BUILD_URL}  

You can find the dependency check report attached.

Best regards,  
Jenkins CI
""",
        subject: "Jenkins Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        to: 'jnikita647@gmail.com'
    )
        }

        failure {
            emailext(
                attachmentsPattern: 'target/dependency-check-report.html',
                body: """Hello,

The Jenkins pipeline **${env.JOB_NAME}** has failed on **Build #${env.BUILD_NUMBER}**.

 **Job Details:**  
- **Job Name:** ${env.JOB_NAME}  
- **Build Number:** ${env.BUILD_NUMBER}  
- **Build URL:** ${env.BUILD_URL}  

Please review the attached logs and reports for more details.

Regards,  
Jenkins CI
""",
                subject: "Jenkins Pipeline Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                to: 'jnikita647@gmail.com'
            )
        }
    }

}
