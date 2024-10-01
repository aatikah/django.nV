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

   stage('OWASP Dependency-Check') {
    steps {
        // Run OWASP Dependency-Check
        dependencyCheck additionalArguments: '''
            -o './'
            -s './'
            -f 'HTML,JSON'
            --prettyPrint''',
            odcInstallation: 'OWASP-Dependency-Check'
    }
    post {
        always {
            // Archive the reports as artifacts
            archiveArtifacts artifacts: 'dependency-check-report.html,dependency-check-report.json', allowEmptyArchive: true

            // Publish HTML report
            publishHTML(target: [
                allowMissing: true,
                alwaysLinkToLastBuild: false,
                keepAll: true,
                reportDir: '.',
                reportFiles: 'dependency-check-report.html',
                reportName: 'Dependency-Check Report'
            ])

            // Parse JSON report to provide a summary
            script {
                if (fileExists('dependency-check-report.json')) {
                    def jsonReport = readJSON file: 'dependency-check-report.json'
                    def vulnerableDependencies = jsonReport.dependencies.findAll { it.vulnerabilities }
                    def totalVulnerabilities = vulnerableDependencies.collect { it.vulnerabilities.size() }.sum()
                    def highVulnerabilities = vulnerableDependencies.collect { 
                        it.vulnerabilities.findAll { vuln -> vuln.cvssv3?.baseScore >= 7.0 }.size()
                    }.sum()

                    echo """OWASP Dependency-Check Summary:
                    Total dependencies scanned: ${jsonReport.dependencies.size()}
                    Dependencies with vulnerabilities: ${vulnerableDependencies.size()}
                    Total vulnerabilities found: ${totalVulnerabilities}
                    High severity vulnerabilities (CVSS >= 7.0): ${highVulnerabilities}
                    """
                } else {
                    echo "Dependency-Check JSON report not found. The scan may have failed."
                }
            }
        }
    }
}
}
}
