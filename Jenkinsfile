pipeline {
    agent {
        label 'slave-jenkins-deb'  // Replace with the label of your slave node
    }
   tools{
       maven 'Maven'
   }
stages{
    stage('Initialize') {
                steps {
                    sh '''
                        echo "PATH = ${PATH}"
                        echo "M2_HOME = ${M2_HOME}"
                    ''' 
                }
            } 
    stage('Echo') {
            steps {
                script {
                    
                    //Remove any existing report file
                    sh 'echo "Hello from Node"'
                    
    
                    
    
                   
               }
            }
            }
    stage('Check Git Secrets With Gitleaks With Error Continue') {
            steps {
                script {
                    
                    //Remove any existing report file
                    sh 'rm -f gitleaks_report.json'
                    
                    // Pull the Gitleaks Docker image
                    sh 'sudo docker pull zricethezav/gitleaks'
                    
                    // Run Gitleaks in a Docker container and capture the exit code
                    def gitleaksStatus = sh(script: 'sudo docker run --rm -v /var/lib/jenkins/workspace/vul-django:/repo zricethezav/gitleaks detect --source /repo --report-path /repo/gitleaks_report.json --report-format json', returnStatus: true)
                    
                   //  Display the Gitleaks report
                    sh 'cat gitleaks_report.json'
                    
                   //  Handle the Gitleaks exit code
                    if (gitleaksStatus == 1) {
                        echo 'Leaks found. Review the Gitleaks report for details.'
                        }
    
                    
    
                   
               }
            }
            }

}
}
