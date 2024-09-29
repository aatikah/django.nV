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
                --report-format=json
            """
            
            def gitleaksResult = sh(script: gitleaksCommand, returnStatus: true)
            
            // Check the exit code
            if (gitleaksResult == 1) {
                echo "Gitleaks found potential secrets. Check the report for details."
                archiveArtifacts artifacts: 'gitleaks-report.json', allowEmptyArchive: true
            } else if (gitleaksResult != 0) {
                error "Gitleaks scan failed with exit code ${gitleaksResult}"
            }
        }
    }
}
   

}
}
