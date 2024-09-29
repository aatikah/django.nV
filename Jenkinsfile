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
  stage('Run Gitleaks') {
    steps {
        script {
            def gitleaksImage = 'zricethezav/gitleaks:v8.15.2'
            def repoPath = '/scan'
            def reportPath = "${WORKSPACE}/gitleaks-report.json"
            
            // Pull the specified gitleaks image
            sh "docker pull ${gitleaksImage}"
            
            // Run gitleaks
            def gitleaksCommand = """
                docker run --rm -v ${WORKSPACE}:${repoPath} \
                ${gitleaksImage} detect \
                --source=${repoPath} \
                --report-path=${repoPath}/gitleaks-report.json \
                --report-format=json \
                --no-git \
                --verbose
            """
            
            // Run gitleaks and capture output
            def (exitCode, output) = sh(script: gitleaksCommand, returnStdout: true, returnStatus: true).tokenize('\n')
            
            echo "Gitleaks command exit code: ${exitCode}"
            echo "Gitleaks output: ${output}"
            
            // Check if the report file exists
            def reportExists = fileExists reportPath
            
            if (reportExists) {
                echo "Gitleaks report found. Archiving..."
                archiveArtifacts artifacts: 'gitleaks-report.json', allowEmptyArchive: false
                
                // Read and log the content of the report
                def reportContent = readFile reportPath
                echo "Report content: ${reportContent}"
                
                if (exitCode == '1') {
                    echo "Gitleaks found potential secrets. Check the report for details."
                } else if (exitCode != '0') {
                    error "Gitleaks scan failed with exit code ${exitCode}"
                }
            } else {
                echo "Gitleaks report not found. Check the output for errors."
                error "Gitleaks scan failed: Report not generated"
            }
        }
    }
}
   

}
}
