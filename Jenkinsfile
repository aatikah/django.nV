pipeline {
    agent any 
   tools{
       maven 'Maven'
   }
    environment {
        REGISTRY = 'https://index.docker.io/v1/'
        REPOSITORY = 'aatikah'
        IMAGE_NAME = 'vul-djangoapp'
        DOCKER_CREDENTIALS_ID = 'docker-credential'
        
        DEFECTDOJO_URL = credentials('DEFECTDOJO_URL')
        GITLEAKS_ENGAGEMENT_ID = '5'
        BANDIT_ENGAGEMENT_ID = '6'
        ZAP_ENGAGEMENT_ID = '7'
        NIKITO_ENGAGEMENT_ID = '8'
        DEFECT_DOJO = 'http://34.170.177.2:8080'
        ARCHERYSEC_URL = 'http://34.170.65.15:8000'
        ARCHERYSEC_PROJECT_ID = '298f50a8-1e2f-4b03-beb6-392398d125b2'
        ARCHERYSEC_USER = 'abuabdillah5444@gmail.com'
        targetUrl = 'http://34.123.8.118'
    
    }
    //DEFECTDOJO_API_KEY = 'DEFECTDOJO_API_KEY'
    
    stages {
        stage('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                ''' 
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

                // Use withCredentials to handle the API key securely
                   // withCredentials([string(credentialsId: 'DEFECTDOJO_API_KEY', variable: 'API_KEY')]) {
                       // def curlCommand = """
                          //  curl -X POST '${DEFECT_DOJO}/api/v2/import-scan/' \
                          //  -H 'Authorization: Token ${API_KEY}' \
                          //  -H 'accept: application/json' \
                         //   -H 'Content-Type: multipart/form-data' \
                         //   -F 'file=@gitleaks_report.json' \
                          //  -F 'scan_type=Gitleaks Scan' \
                          //  -F 'engagement=${GITLEAKS_ENGAGEMENT_ID}' \
                         //   -F 'product_name=django-pipeline'
                      //  """
                        //sh curlCommand
                       // def response = sh(script: curlCommand, returnStdout: true).trim()
                        //echo "Response from DefectDojo: ${response}"
                   // }

                // Use withCredentials to handle the API key securely
                    withCredentials([string(credentialsId: 'DEFECTDOJO_API_KEY', variable: 'API_KEY')]) {
                    def response = sh(
                        script: """
                        curl -X POST '${DEFECT_DOJO}/api/v2/import-scan/' \
                        -H 'Authorization: Token $API_KEY' \
                        -H 'accept: application/json' \
                        -H 'Content-Type: multipart/form-data' \
                        -F 'file=@gitleaks_report.json' \
                        -F 'scan_type=Gitleaks Scan' \
                        -F 'engagement=${GITLEAKS_ENGAGEMENT_ID}' \
                        -F 'product_name=django-pipeline'
                            """,
                            returnStdout: true
                        ).trim()

                        echo "Response from DefectDojo: ${response}"
                   
                }
           }
        }
        }

        
        
        //stage('Check Git Secrets With Gitleaks') {
            //steps {
                // Remove any existing report file
               // sh 'rm gitleaks_report.json || true'
                
                // Pull the Gitleaks Docker image
               // sh 'sudo docker pull zricethezav/gitleaks'
                
                // Run Gitleaks in a Docker container
                //sh 'sudo docker run --rm -v /var/lib/jenkins/workspace/vul-django:/repo zricethezav/gitleaks detect --source /repo --exit-code 1 --report-path /repo/gitleaks_report.json --report-format json'
                //sh 'sudo docker run --rm -v /var/lib/jenkins/workspace/vul-django:/repo zricethezav/gitleaks detect --source /repo --report-path /repo/gitleaks_report.json --report-format json'
                
                // Display the Gitleaks report
               // sh 'cat gitleaks_report.json'
           // }
        //}

       // stage('Source Composition Analysis'){
           // steps{
               // script{
                  //  sh 'rm owasp* || true'
                  //  sh 'wget "https://raw.githubusercontent.com/aatikah/django.nV/master/owasp-dependency-check.sh"'
                 //   sh 'chmod +x owasp-dependency-check.sh'
                 //   sh 'bash owasp-dependency-check.sh'
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
              //  }
          //  }
       // }

    

      // stage('SAST With Sonarqube') {
        //steps {
           //withSonarQubeEnv('vul-django') {
               // withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                  //  sh '''
                    //export SONAR_TOKEN=${SONAR_TOKEN}
                  //  mvn sonar:sonar -Dsonar.token=$SONAR_TOKEN
                 //   '''
           // }
        //}
   // }
//}

        //stage('SAST With Sonarqube') {
           // steps {
               // checkout([
                //    $class: 'GitSCM',
               //     branches: [[name: '*/master']],
               //     userRemoteConfigs: [[url: 'https://github.com/aatikah/django.nV.git']]
              //  ])
               // withSonarQubeEnv('vul-django') {
                   // withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                       // sh '''
                         //   export SONAR_TOKEN=${SONAR_TOKEN}
                        //    mvn sonar:sonar -Dsonar.token=$SONAR_TOKEN
                   //     '''
           // }
      //  }
  //  }
//}

       //stage('SAST with Bandit') {
           // steps {
              // script {
                // Remove any existing Bandit report file
               // sh 'rm -f bandit_report.html'
                    
                // Ensure Bandit is installed in the Jenkins environment
               // sh 'pip install bandit'
                
                // Run Bandit on the code directory
                  //  sh '''
                  //  /home/abuabdillah5444/myenv/bin/bandit -r /var/lib/jenkins/workspace/vul-django -o bandit_report.html -f html
                //  '''
                //sh 'bandit -r /var/lib/jenkins/workspace/vul-django -o bandit_report.html -f html'
                
                // Display the Bandit report
                   // sh 'cat bandit_report.html'

                // Assuming /var/www/html/bandit_reports is served by your web server
                   // sh 'cp bandit_report.html /var/www/html/bandit_reports/'
                    
                // Archive the Bandit report as an artifact
               // archiveArtifacts artifacts: 'bandit_report.html', allowEmptyArchive: true

            // Publish the report using HTML Publisher Plugin
            //publishHTML(target: [
               // reportDir: '',
              //  reportFiles: 'bandit_report.html',
               // reportName: 'Bandit Security Report',
              //  keepAll: true,
              //  alwaysLinkToLastBuild: true
           // ])
        //}
               // echo "Report available at: http://34.28.86.41/bandit_reports/bandit_report.html"
   // }
//}

        stage('SAST with Bandit') {
            steps {
                script {
                    def banditHtmlStatus = sh(
                        script: '/home/abuabdillah5444/myenv/bin/bandit -r /var/lib/jenkins/workspace/vul-django -o bandit_report.html -f html',
                        returnStatus: true
                    )

                    def banditJsonStatus = sh(
                        script: '/home/abuabdillah5444/myenv/bin/bandit -r /var/lib/jenkins/workspace/vul-django -o bandit_report.json -f json',
                        returnStatus: true
                    )
        
                    if (banditHtmlStatus != 0) {
                        echo "Bandit found issues in the code. Please review the report at bandit_report.html."
                        // Optionally, you can fail the build if you want to treat these issues as critical
                        // error("Bandit scan failed with exit code ${banditStatus}")
                    }
                    
                    // Display the Bandit report
                        sh 'cat bandit_report.html'
                        sh 'cat bandit_report.json'

                     // Archive the Bandit report as an artifact
                    archiveArtifacts artifacts: 'bandit_report.html', allowEmptyArchive: true

                // Publish the report using HTML Publisher Plugin
                publishHTML(target: [
                    reportDir: '',
                    reportFiles: 'bandit_report.html',
                    reportName: 'Bandit Security Report',
                    keepAll: true,
                    alwaysLinkToLastBuild: true
                ])

                    // Use withCredentials to handle the API key securely
                    withCredentials([string(credentialsId: 'DEFECTDOJO_API_KEY', variable: 'API_KEY')]) {
                    def response = sh(
                        script: """
                        curl -X POST '${DEFECT_DOJO}/api/v2/import-scan/' \
                        -H 'Authorization: Token $API_KEY' \
                        -H 'accept: application/json' \
                        -H 'Content-Type: multipart/form-data' \
                        -F 'file=@bandit_report.json' \
                        -F 'scan_type=Bandit Scan' \
                        -F 'engagement=${BANDIT_ENGAGEMENT_ID}' \
                        -F 'product_name=django-pipeline'
                            """,
                            returnStdout: true
                        ).trim()

                        echo "Response from DefectDojo: ${response}"

                       // withCredentials([string(credentialsId: 'DEFECTDOJO_API_KEY', variable: 'API_KEY')]) {
                      //  def curlCommand = """
                         //   curl -X POST '${DEFECT_DOJO}/api/v2/import-scan/' \
                        //    -H 'Authorization: Token $API_KEY' \
                          //  -H 'accept: application/json' \
                        //    -H 'Content-Type: multipart/form-data' \
                          //  -F 'file=@bandit_report.json' \
                          //  -F 'scan_type=Bandit Scan' \
                          //  -F 'engagement=${BANDIT_ENGAGEMENT_ID}' \
                          //  -F 'product_name=django-pipeline'
                      //  """
                        //sh curlCommand
                        //def response = sh(script: curlCommand, returnStdout: true).trim()
                        //echo "Response from DefectDojo: ${response}"

                    }
                }
            }
        }


        
        stage('Build Docker Image') {

            steps {
                script {
                    dockerImage = docker.build("${REPOSITORY}/${IMAGE_NAME}")
                }
            }
        }

        
         stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("${REGISTRY}", DOCKER_CREDENTIALS_ID) {
                        dockerImage.push('v1')
                    }
                }
            }
        }
        stage('Deploy to GCP VM') {
            steps {
                sshagent(['tomcatkey']) {
                sh '''
                ssh -o StrictHostKeyChecking=no abuabdillah5444@34.123.8.118 "sudo docker pull aatikah/vul-djangoapp:v1 && sudo docker run -d -p 8000:8000 aatikah/vul-djangoapp:v1"
                '''
    }
    }
}
  //  stage('DAST With OWASP ZAP Scan') {
           // steps {
               // script {
                   

                    // Allow ZAP to start up
                   // sleep 10

                    //sh 'sudo docker run -t zaproxy/zap-stable zap-baseline.py -t http://34.135.208.39 -J zap_report.json'
                    
                  // sh '''
                       // sudo chmod -R 777 /var/lib/jenkins/workspace/vul-django
                       // sudo docker run -v /var/lib/jenkins/workspace/vul-django:/zap/wrk -t zaproxy/zap-stable zap-baseline.py -t http://34.123.8.118 -J zap_report.json || true
                     //   echo "Sleeping for 20 seconds"
                   //     sleep 20
                   //  '''
                 //   sh 'cat zap_report.json || true'

                    
                   
                   // sh '''
                     //  sudo chmod -R 777 /var/lib/jenkins/workspace/vul-django
                      //  sudo docker run -v /var/lib/jenkins/workspace/vul-django:/zap/wrk -t zaproxy/zap-stable zap-baseline.py -t http://34.123.8.118 -r zap_report.html ||true
                     //   echo "Sleeping for 10 seconds"
                     //   sleep 20
                  //  '''
                  //  sh 'cat zap_report.html || true'
                   

                    // Copy the ZAP JSON report out of the Docker container
                   // sh 'sudo docker cp zap:/zap/zap_report.json ./zap_report.json'

                    // Copy the ZAP HTML report out of the Docker container
                   // sh 'sudo docker cp zap:/zap/zap_report.html ./zap_report.html'

                    // Use the Jenkins HTML Publisher Plugin to display the report
                     //   publishHTML(target: [
                         //   reportDir: '.',
                         //   reportFiles: 'zap_report.html',
                         //   reportName: 'OWASP ZAP Report'
                       // ])
                    
                        // Securely use DefectDojo API Key within a credentials block
                    //withCredentials([string(credentialsId: 'DEFECTDOJO_API_KEY', variable: 'API_KEY')]) {
                       // def response = sh(
                         //   script: """
                         //   curl -X POST '${DEFECT_DOJO}/api/v2/import-scan/' \
                            //-H 'Authorization: Token $API_KEY' \
                           // -H 'accept: application/json' \
                           // -H 'Content-Type: multipart/form-data' \
                          //  -F 'file=@zap_report.json' \
                          //  -F 'scan_type=ZAP Scan' \
                          //  -F 'engagement=${ZAP_ENGAGEMENT_ID}' \
                          //  -F 'product_name=django-pipeline'
                          //  """,
                          //  returnStdout: true
                        //).trim()

                        //echo "Response from DefectDojo: ${response}"

                      //  withCredentials([string(credentialsId: 'DEFECTDOJO_API_KEY', variable: 'API_KEY')]) {
                      // def curlCommand = """
                          //  curl -X POST '${DEFECT_DOJO}/api/v2/import-scan/' \
                         //  -H 'Authorization: Token $API_KEY' \
                         //   -H 'accept: application/json' \
                        //    -H 'Content-Type: multipart/form-data' \
                          //  -F 'file=@zap_report.json' \
                           // -F 'scan_type=ZAP Scan' \
                          //  -F 'engagement=${ZAP_ENGAGEMENT_ID}' \
                          //  -F 'product_name=django-pipeline'
                        //"""
                        //sh curlCommand
                        //def response = sh(script: curlCommand, returnStdout: true).trim()
                        //echo "Response from DefectDojo: ${response}"
                   // }

                    // Send ZAP scan results to ArcherySec
                    //withCredentials([string(credentialsId: 'archerysec', variable: 'ARCHERYSEC_API_KEY')]) {
                       // def archerysecCommand = """
                        //    curl -X POST '${ARCHERYSEC_URL}/api/v1/uploadscan/' \
                           // -H 'Authorization: ${ARCHERYSEC_API_KEY}' \
                           // -H 'Content-Type: multipart/form-data' \
                           // -F 'scanner=zap_scan' \
                           // -F 'file=@zap_report.json' \
                           // -F 'scan_url=http://34.132.231.218' \
                         //   -F 'project_id=${ARCHERYSEC_PROJECT_ID}' \
                          //  -F 'username=${ARCHERYSEC_USER}'
                      //  """
                        //sh archerysecCommand
                    //}

                   // sh '''
                   // rm -rf archerysec_env
                   // python3 -m venv archerysec_env
                   // . archerysec_env/bin/activate
                  //  pip install archerysec-cli
                //    mkdir -p /tmp/archerysec-scans-report
                  //  archerysec-cli -h http://34.170.65.15:8000 -t cHnQc3bpwLV3sMfiRAj2jLr42O_fGkyvYmt11KY7GD8Tjv5CbYWlG0Dqps49tDcq --cicd_id=032ce5e4-0d26-4a28-af1d-c53d512e644b --project=298f50a8-1e2f-4b03-beb6-392398d125b2 --zap-base-line-scan --report_path=/tmp/archerysec-scans-report/
                 // '''
                    //sh 'archerysec-cli -h http://34.170.65.15:8000 -t cHnQc3bpwLV3sMfiRAj2jLr42O_fGkyvYmt11KY7GD8Tjv5CbYWlG0Dqps49tDcq --cicd_id=032ce5e4-0d26-4a28-af1d-c53d512e644b --project=298f50a8-1e2f-4b03-beb6-392398d125b2 --zap-base-line-scan --report_path=/tmp/archerysec-scans-report/ --verbose || true'
                    

                   // def zapReport = readFile 'zap_report.html'
                  //  def response = httpRequest(
                      //  url: 'http://34.170.65.15:8000/api/v1/zap-scan/',
                      //  requestBody: zapReport,
                      //  httpMode: 'POST',
                      //  contentType: 'APPLICATION_JSON',
                      //  customHeaders: [
                        //    [name: 'Authorization', value: 'cHnQc3bpwLV3sMfiRAj2jLr42O_fGkyvYmt11KY7GD8Tjv5CbYWlG0Dqps49tDcq']
                      //  ]
                   // )
                  //  if (response.status == 201) {
                   //     println 'ZAP report exported to ArcherySec successfully'
                  //  } else {
                   //     error "Failed to export ZAP report to ArcherySec: ${response.status} - ${response.content}"
                   // }
                    
                
            //}
       // }
        
        stage('DAST With NIKITO Scan') {
            steps {
                script {
                    

                    // Run the Nikto scan command
                    //sh '/home/abuabdillah5444/nikto/program/nikto.pl -h ${targetUrl} -o nikto_scan_results.html || true'
                    sh "/home/abuabdillah5444/nikto/program/nikto.pl -h '${targetUrl}' -o nikto_scan_results.html || true"
                    sh 'cat nikto_scan_results.html'
                    
                   // sh '/home/abuabdillah5444/nikto/program/nikto.pl -h ${targetUrl} -o nikto_scan_results.json || true'
                    sh "/home/abuabdillah5444/nikto/program/nikto.pl -h '${targetUrl}' -o nikto_scan_results.json || true"
                    sh 'cat nikto_scan_results.json'

                    // Use the Jenkins HTML Publisher Plugin to display the report
                        publishHTML(target: [
                            reportDir: '.',
                            reportFiles: 'nikto_scan_results.html',
                            reportName: 'NIKITO Scan Report'
                        ])

                    // Use withCredentials to handle the API key securely
                    withCredentials([string(credentialsId: 'DEFECTDOJO_API_KEY', variable: 'API_KEY')]) {
                    def response = sh(
                        script: """
                        curl -X POST '${DEFECT_DOJO}/api/v2/import-scan/' \
                        -H 'Authorization: Token $API_KEY' \
                        -H 'accept: application/json' \
                        -H 'Content-Type: multipart/form-data' \
                        -F 'file=@nikto_scan_results.json' \
                        -F 'scan_type=Nikito Scan' \
                        -F 'engagement=${NIKITO_ENGAGEMENT_ID}' \
                        -F 'product_name=django-pipeline'
                            """,
                            returnStdout: true
                        ).trim()

                        echo "Response from DefectDojo: ${response}"
                }
            }
        }
    }
    
    }

}

