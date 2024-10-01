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

    stage('Source Composition Analysis - OWASP Dependency-Check') {
            steps {
                script {
                    // Catch vulnerabilities but continue the build process
                    catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                        dependencyCheck additionalArguments: '--format ALL'
                    }
            }
            post {
                always {
                    // Archive the report files (HTML and JSON)
                    archiveArtifacts artifacts: 'dependency-check-report/*.json, dependency-check-report/*.html', allowEmptyArchive: true

                    // Publish the HTML report for easy viewing in Jenkins
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'dependency-check-report',
                        reportFiles: 'dependency-check-report.html',
                        reportName: 'OWASP Dependency-Check HTML Report'
                    ])
                }
            }
        }

  
   

}
}
}
