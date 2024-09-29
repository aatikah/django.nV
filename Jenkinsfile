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
            def gitleaksImage = 'zricethezav/gitleaks:latest'
            def repoPath = '/scan'  // Mount point inside the container
            
            // Pull the latest gitleaks image
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
            def gitleaksOutput = sh(script: gitleaksCommand, returnStdout: true)
            echo "Gitleaks output: ${gitleaksOutput}"
            
            // Check if the report file exists
            def reportExists = fileExists 'gitleaks-report.json'
            
            if (reportExists) {
                echo "Gitleaks report found. Archiving..."
                archiveArtifacts artifacts: 'gitleaks-report.json', allowEmptyArchive: false
            } else {
                echo "Gitleaks report not found. Check the output for errors."
                error "Gitleaks scan failed: Report not generated"
            }
        }
    }
}
   

}
}
