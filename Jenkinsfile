pipeline {
    agent {
        label 'slave-jenkins-deb'  // Replace with the label of your slave node
    }
   
stages{
    
    stage('Echo') {
            steps {
                script {
                    
                    
                    sh 'echo "Hello from Node"'
                    
                   
               }
            }
            }
 stage('Run Gitleaks with Custom Config') {
            steps {
                script {
                    // Pull and run the Gitleaks Docker image with a custom config file
                    sh '''
                        docker run --rm -v $(pwd):/path -v $(pwd)/.gitleaks.toml:/.gitleaks.toml zricethezav/gitleaks:latest detect --source /path --config /.gitleaks.toml --report-format json --report-path /path/gitleaks-report.json || true
                        '''
                    // Archive the reports as artifacts
                        archiveArtifacts artifacts: 'gitleaks-report.json', allowEmptyArchive: true

            // Publish HTML report
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: '.',
                        reportFiles: 'bandit-report.html',
                        reportName: 'Bandit Security Scan Report'
                    ])

                }
                 // Display the contents of the report in a separate step
                //script {
                  //  echo "Gitleaks Report:"
                  //  sh 'cat gitleaks-report.json || echo "Report not found or empty."'
                //}
            }
        }
     stage('Source Composition Analysis'){
            steps{
                script{
                    sh 'rm owasp* || true'
                    sh 'wget "https://raw.githubusercontent.com/aatikah/django.nV/master/owasp-dependency-check.sh"'
                   // sh 'chmod +x owasp-dependency-check.sh'
                    sh 'bash owasp-dependency-check.sh'
                    //sh 'cat /home/jenkins/workspace/django/report/dependency-check-report.xml'
                    //sh 'cat /home/jenkins/workspace/django/report/dependency-check-report.json'

                    // Archive the reports as artifacts
                        archiveArtifacts artifacts: '/report/dependency-check-report.json,/report/dependency-check-report.html,/report/dependency-check-report.xml', allowEmptyArchive: true

            // Publish HTML report
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: '/report',
                        reportFiles: 'dependency-check-report.html',
                        reportName: 'OWASP Dependency Checker Report'
                    ])

                
                }
            }
        }

   
}
}
