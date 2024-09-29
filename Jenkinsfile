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
            def gitleaksImage = 'zricethezav/gitleaks:v8.15.2'  // Specify a known working version
            def repoPath = '/scan'  // Mount point inside the container
            
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
            def gitleaksOutput = sh(script: gitleaksCommand, returnStdout: true, returnStatus: true)
            
            echo "Gitleaks command exit code: ${gitleaksOutput[0]}"
            echo "Gitleaks output: ${gitleaksOutput[1]}"
            
            // Check if the report file exists
            def reportExists = fileExists 'gitleaks-report.json'
            
            if (reportExists) {
                echo "Gitleaks report found. Archiving..."
                archiveArtifacts artifacts: 'gitleaks-report.json', allowEmptyArchive: false
                
                // Read and log the content of the report
                def reportContent = readFile 'gitleaks-report.json'
                echo "Report content: ${reportContent}"
                
                if (gitleaksOutput[0] == 1) {
                    echo "Gitleaks found potential secrets. Check the report for details."
                } else if (gitleaksOutput[0] != 0) {
                    error "Gitleaks scan failed with exit code ${gitleaksOutput[0]}"
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
