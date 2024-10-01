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
     stage('Source Composition Analysis'){
            steps{
                script{
                    sh 'rm owasp* || true'
                    sh 'wget "https://raw.githubusercontent.com/aatikah/django.nV/master/owasp-dependency-check.sh"'
                  //  sh 'chmod +x owasp-dependency-check.sh'
                    sh 'bash owasp-dependency-check.sh'
                 //   sh 'cat /var/lib/jenkins/workspace/vul-django/odc-reports/dependency-check-report.xml'
                   // sh 'cat /var/lib/jenkins/workspace/vul-django/odc-reports/dependency-check-report.json'

                    //def response = sh(
                      //  script: """
                      //  curl -X POST ${DEFECT_DOJO}/api/v2/import-scan/ \
                      //  -H "Authorization: Token ${DEFECTDOJO_API_KEY}" \
                      //  -H "accept: application/json" \
                       // -H "Content-Type: multipart/form-data" \
                       // -F "file=@/var/lib/jenkins/workspace/vul-django/odc-reports/dependency-check-report.json" \
                      //  -F 'scan_type=Dependency Check Scan' \
                      //  -F 'engagement=${ENGAGEMENT_ID}' \
                      //  -F 'product_name=django-pipeline'
                      //  """,
                    //    returnStdout: true
                  //  ).trim()

                  //  echo "Response from DefectDojo: ${response}"
                }
            }
        }

   
}
}
