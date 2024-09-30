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

                }
                 // Display the contents of the report in a separate step
                //script {
                  //  echo "Gitleaks Report:"
                  //  sh 'cat gitleaks-report.json || echo "Report not found or empty."'
                //}
            }
        }

    stage('Bandit Security Scan') {
    steps {
        script {
            // Install Bandit if not already installed
            sh 'pip install bandit'
            
            // Run Bandit scan and generate reports
            sh '''
                python3 -m venv bandit_venv
                . bandit_venv/bin/activate
                pip install --upgrade pip
                pip install bandit
                
            
                bandit -r . -f json -o bandit-report.json --exit-zero
                bandit -r . -f html -o bandit-report.html --exit-zero

                deactivate
            '''
            
            // Archive the reports as artifacts
            archiveArtifacts artifacts: 'bandit-report.json,bandit-report.html', allowEmptyArchive: true
        }
    }
    post {
        always {
            // Publish HTML report
            publishHTML(target: [
                allowMissing: false,
                alwaysLinkToLastBuild: false,
                keepAll: true,
                reportDir: '.',
                reportFiles: 'bandit-report.html',
                reportName: 'Bandit Security Scan Report'
            ])
            
            // Parse JSON report to check for issues
            script {
                def jsonReport = readJSON file: 'bandit-report.json'
                def issueCount = jsonReport.results.size()
                if (issueCount > 0) {
                    echo "Bandit found ${issueCount} potential security issue(s). Please review the report."
                } else {
                    echo "Bandit scan completed successfully with no issues found."
                }
            }
        }
    }
}
   

}
}
