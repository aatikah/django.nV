pipeline {
    agent {
        label 'slave-jenkins-deb'  // Replace with the label of your slave node
    }
    environment{
        DOCKER_REGISTRY = 'https://index.docker.io/v1/'
        DOCKER_IMAGE = 'aatikah/django-app'
        remoteHost = '34.134.182.0'
        DEFECTDOJO_API_KEY = credentials('DEFECTDOJO_API_KEY')
        DEFECTDOJO_URL = 'http://34.42.127.145:8080'
        PRODUCT_NAME = 'django-project'
        ENGAGEMENT_NAME = '1'
        
    }
     

stages{
    
    stage('Testing Node') {
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
                    // Archive the reports as artifacts
                        archiveArtifacts artifacts: 'gitleaks-report.json', allowEmptyArchive: true

            // Publish HTML report
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: '.',
                        reportFiles: 'bandit-report.html',
                        reportName: 'Bandit Security Scan Report'
                    ])

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
                   // sh 'chmod +x owasp-dependency-check.sh'
                    sh 'bash owasp-dependency-check.sh'
                    //sh 'cat /home/jenkins/workspace/django/report/dependency-check-report.xml'
                    //sh 'cat /home/jenkins/workspace/django/report/dependency-check-report.json'

                    // Archive the reports as artifacts
                        archiveArtifacts artifacts: 'report/dependency-check-report.json,report/dependency-check-report.html,report/dependency-check-report.xml', allowEmptyArchive: true

            // Publish HTML report
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: './report',
                        reportFiles: 'dependency-check-report.html',
                        reportName: 'OWASP Dependency Checker Report'
                    ])

                    // Parse JSON report to check for issues
           // This if block can be added in another script block outside this script block to fail pipeline if cvssv is above 7
                if (fileExists('report/dependency-check-report.json')) {
                    def jsonReport = readJSON file: 'report/dependency-check-report.json'
                    def vulnerabilities = jsonReport.dependencies.collect { it.vulnerabilities ?: [] }.flatten()
                    def highVulnerabilities = vulnerabilities.findAll { it.cvssv3?.baseScore >= 7 }
                    echo "OWASP Dependency-Check found ${vulnerabilities.size()} vulnerabilities, ${highVulnerabilities.size()} of which are high severity (CVSS >= 7.0)"
                } else {
                    echo "Dependency-Check JSON report not found. The scan may have failed."
                }
            

                
                }
            }
        }

    //BANDIT STAGE
    stage('SAST With Bandit Security Scan') {
    steps {
        script {
           
            // Run Bandit scan and generate reports
            sh '''
                python3 -m venv bandit_venv
                . bandit_venv/bin/activate
                pip install --upgrade pip
                pip install bandit
                
            
                bandit -r . -f json -o bandit-report.json --exit-zero
                bandit -r . -f html -o bandit-report.html --exit-zero

                deactivate
            '''
            
            // Archive the reports as artifacts
            archiveArtifacts artifacts: 'bandit-report.json,bandit-report.html', allowEmptyArchive: true

            // Publish HTML report
            publishHTML(target: [
                allowMissing: false,
                alwaysLinkToLastBuild: false,
                keepAll: true,
                reportDir: '.',
                reportFiles: 'bandit-report.html',
                reportName: 'Bandit Security Scan Report'
            ])
            
            // Parse JSON report to check for issues
            script {
                def jsonReport = readJSON file: 'bandit-report.json'
                def issueCount = jsonReport.results.size()
                if (issueCount > 0) {
                    echo "Bandit found ${issueCount} potential security issue(s). Please review the report."
                } else {
                    echo "Bandit scan completed successfully with no issues found."
                }
            }
        }
    }
    
}
    

    stage('Build and Push Docker Image') {

            steps {
                script {
                    // Wrap the Docker commands with withCredentials to securely access the Docker credentials
                    withCredentials([usernamePassword(credentialsId: 'DOCKER_CREDENTIALS_ID', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        // Build the Docker image
                        sh "docker build -t ${DOCKER_IMAGE} ."
                        //sh "docker build -t ${DOCKER_IMAGE}:v5 ."

                        // Log in to the Docker registry using a more secure method. set +x set -x This turns off command echoing temporarily
                        sh '''
                            set +x
                            echo "$DOCKER_PASSWORD" | docker login $DOCKER_REGISTRY -u "$DOCKER_USERNAME" --password-stdin
                            set -x
                        '''
                       
                        // Push the Docker image
                        //sh "docker push ${DOCKER_IMAGE}:v5"
                        sh "docker push ${DOCKER_IMAGE}"
                        
                        // Log out from the Docker registry
                        sh "docker logout $DOCKER_REGISTRY"

                        // Clean up: remove any leftover Docker credentials
                        sh "rm -f /home/jenkins/.docker/config.json"
                    }
                }
            }
        }

    stage('Deploy to GCP VM') {
    steps {
        script {
            //def remoteHost = '34.134.182.0'
            def remoteUser = 'jenkins-slave'
            def dockerImage = 'aatikah/django-app'
            
            sshagent(['slave-jenkins-key']) {
                // Stop and remove the old container if it exists
                sh """
                    ssh -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} '
                        container_id=\$(docker ps -q --filter ancestor=${dockerImage})
                        if [ ! -z "\$container_id" ]; then
                            docker stop \$container_id
                            docker rm \$container_id
                        fi
                    '
                """
                
                // Pull the latest image and run the new container
                sh """
                    ssh -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} '
                        docker pull ${dockerImage} && 
                        docker run -d --restart unless-stopped -p 8000:8000 --name my-django-app ${dockerImage}
                    '
                """
                
                // Verify the deployment
                sh """
                    ssh -o StrictHostKeyChecking=no ${remoteUser}@${remoteHost} '
                        if docker ps | grep -q ${dockerImage}; then
                            echo "Deployment successful"
                        else
                            echo "Deployment failed"
                            exit 1
                        fi
                    '
                """
            }
        }
    }
}

    stage('DAST OWASP ZAP Scan') {
    steps {
        script {
            def zapHome ='/opt/zaproxy' // Path to ZAP installation
            //def targetURL = 'http://34.134.182.0'  // Update this to your application's URL
            def reportNameHtml = "zap-scan-report.html"
            //def reportNameJson = "zap-scan-report.json"
            
            // Perform ZAP scan
            sh """
                
                ${zapHome}/zap.sh -cmd \
                    -quickurl ${remoteHost} \
                    -quickprogress 

            """
            
            // Archive the ZAP reports
            //archiveArtifacts artifacts: "${reportNameHtml},${reportNameJson}", fingerprint: true
            archiveArtifacts artifacts: "${reportNameHtml}", fingerprint: true


            // Read and parse the HTML report
            def htmlReportContent = readFile(reportNameHtml)

            // Example: Check for high alerts in HTML using regex (customize based on your report structure)
            def highRiskPattern = ~/<span class="risk"><strong>High<\/strong><\/span>.*?<a href="(.*?)">(.*?)<\/a>/
            def highAlerts = []

            htmlReportContent.eachMatch(highRiskPattern) { match ->
                highAlerts.add([url: match[1], alert: match[2]])
            }
            
            if (highAlerts.size() > 0) {
                echo "Found ${highAlerts.size()} high-risk vulnerabilities!"
                highAlerts.each { alert ->
                    echo "High Risk Alert: ${alert.alert} at ${alert.url}"
                }
                // Exit with code 1 if high-risk vulnerabilities are found
                error "OWASP ZAP scan found high-risk vulnerabilities. Check the ZAP report for details."
            }else {
                echo "No high-risk vulnerabilities found."
            }
            
            // Read and parse JSON report
           // def zapJson = readJSON file: reportNameJson
            
            // Example: Check for high alerts in JSON
           // def highAlerts = zapJson.site[0].alerts.findAll { it.riskcode >= 3 }
            
          //  if (highAlerts.size() > 0) {
            //    echo "Found ${highAlerts.size()} high-risk vulnerabilities!"
              //  highAlerts.each { alert ->
                //    echo "High Risk Alert: ${alert.alert} at ${alert.url}"
               // }
              //  error "OWASP ZAP scan found high-risk vulnerabilities. Check the ZAP report for details."
           // }

             // Publish HTML report
            publishHTML(target: [
                allowMissing: false,
                alwaysLinkToLastBuild: false,
                keepAll: true,
                reportDir: '.',
                reportFiles: 'zap-scan-report.html',
                reportName: "ZAP Security Report"
            ])
            
            // Optional: Publish JSON report as a build artifact
            //archiveArtifacts artifacts: 'zap-scan-report.json', fingerprint: true

            
        }
    }
   
           
     
}

    stage('DAST with Nikto') {
    steps {
        script {

            //def TARGET_URL = 'http://34.134.182.0'
            // Run Nikto scan
            sh """
                /home/abuabdillah5444/nikto/program/nikto.pl -h ${remoteHost} -output nikto_output.json -Format json
                /home/abuabdillah5444/nikto/program/nikto.pl -h ${remoteHost} -output nikto_output.html -Format html
            """
            
            // Archive the results
            archiveArtifacts artifacts: 'nikto_output.*', allowEmptyArchive: true
            
            // Optional: Publish HTML report
            publishHTML(target: [
                allowMissing: false,
                alwaysLinkToLastBuild: false,
                keepAll: true,
                reportDir: '.',
                reportFiles: 'nikto_output.html',
                reportName: 'Nikto DAST Report'
            ])
        }
    }
}

    stage('Forward Reports to DefectDojo') {
            steps {
                script {
                    // Install Python requests library if not already available
                    sh '''
                    python3 -m venv venv
                    bash -c "source venv/bin/activate && pip install requests"
                    '''

                    // Function to upload a report to DefectDojo
                    def uploadToDefectDojo = { reportPath, reportType ->
                        def scriptContent = """
import requests
import json

url = "${DEFECTDOJO_URL}/api/v2/import-scan/"
headers = {
    'Authorization': 'Token ${DEFECTDOJO_API_KEY}',
    'Accept': 'application/json'
}
data = {
    'product_name': '${PRODUCT_NAME}',
    'engagement_name': '${ENGAGEMENT_NAME}',
    'scan_type': '${reportType}',
    'active': 'true',
    'verified': 'true',
}
files = {'file': open('${reportPath}', 'rb')}

response = requests.post(url, headers=headers, data=data, files=files)
print(response.text)
if response.status_code != 201:
    raise Exception(f"Failed to upload {reportType} report to DefectDojo: {response.text}")
"""
                        writeFile file: 'upload_to_defectdojo.py', text: scriptContent
                        sh 'python3 upload_to_defectdojo.py'
                    }

                    // Upload Gitleaks report
                    uploadToDefectDojo('gitleaks-report.json', 'Gitleaks Scan')

                    // Upload OWASP Dependency Check report
                    uploadToDefectDojo('report/dependency-check-report.json', 'Dependency Check Scan')

                    // Upload Bandit report
                    uploadToDefectDojo('bandit-report.json', 'Bandit Scan')

                    // Upload ZAP report
                    //uploadToDefectDojo('zap-scan-report.html', 'ZAP Scan')

                    // Upload Nikto report
                    uploadToDefectDojo('nikto_output.json', 'Nikto Scan')
                }
            }
        }


   
}
}
