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
                    // Pull and run the Gitleaks Docker image with default config
                    sh '''
                    docker run --rm -v $(pwd):/path zricethezav/gitleaks:latest detect --source /path --no-config --report-format json --report-path gitleaks-report.json
                    '''
                }
            }
        }
   

}
}
