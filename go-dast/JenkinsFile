pipeline {
    agent any

    stages {
        stage('test') {
            steps {
                sh 'zap.sh -cmd -quickurl http://98.81.247.138:8080/api/v1/employee/health -quickprogress -quickout /var/lib/jenkins/out2.html'
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
        }

        success {
            emailext(
        body: """Hello,

The Jenkins pipeline *${env.JOB_NAME}* has completed successfully on *Build #${env.BUILD_NUMBER}*.

*Build Details:*  
- *Job Name:* ${env.JOB_NAME}  
- *Build Number:* ${env.BUILD_NUMBER}  
- *Build URL:* ${env.BUILD_URL}  

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
                body: """Hello,

The Jenkins pipeline *${env.JOB_NAME}* has failed on *Build #${env.BUILD_NUMBER}*.

 *Job Details:*  
- *Job Name:* ${env.JOB_NAME}  
- *Build Number:* ${env.BUILD_NUMBER}  
- *Build URL:* ${env.BUILD_URL}  

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
