node {
    try {
        stage('Test') {
            sh 'zap.sh -cmd -quickurl http://54.82.9.247:8080/employee/health -port 8090 -quickprogress -quickout /var/lib/jenkins/out2.html'
        }

        stage('Publish') {
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

        echo 'Pipeline completed.'
        

        // Send success email
        emailext(
            body: """Hello,

The Jenkins pipeline **${env.JOB_NAME}** has completed successfully on **Build #${env.BUILD_NUMBER}**.

**Build Details:**  
- **Job Name:** ${env.JOB_NAME}  
- **Build Number:** ${env.BUILD_NUMBER}  
- **Build URL:** ${env.BUILD_URL}  


Best regards,  
Jenkins CI
""",
            subject: "Jenkins Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
            to: 'jnikita647@gmail.com'
        )

    } catch (Exception e) {
        echo 'Pipeline failed.'

        // Send failure email
        emailext(
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
        throw e
    }
}
