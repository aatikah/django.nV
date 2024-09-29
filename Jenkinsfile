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
    
     stage('Prepare Repository') {
    steps {
        git url: 'https://github.com/aatikah/django.nV.git'
    }
}
    
    stage('Check Git Secrets With Gitleaks With Error Continue') {
            steps {
                script {

                    // Clone the repository
                   // checkout scm
                    
                    //Remove any existing report file
                    sh 'rm -f gitleaks_report.json'
                    
                    // Pull the Gitleaks Docker image
                    sh 'docker pull zricethezav/gitleaks:latest'
                    
                    // Run Gitleaks in a Docker container and capture the exit code
                    def gitleaksStatus = sh(script: 'docker run --rm -v /var/lib/jenkins/workspace/django:/report zricethezav/gitleaks:latest detect --source /report --report-path /report/gitleaks_report.json --report-format json', returnStatus: true)
                    
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
