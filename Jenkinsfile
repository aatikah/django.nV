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
        ENGAGEMENT_ID = '1'
        
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
            def reportNameHtml = "${WORKSPACE}/zap-scan-report.html"
            //def reportNameXml = "zap-scan-report.xml"
            
            // Perform ZAP scan
            //sh """
              //  ${zapHome}/zap.sh -cmd \
               //     -quickurl ${remoteHost} \
               //     -quickprogress \
               //     -quickout ${reportNameHtml}    
           // """
            def zapOutput = sh(script: """
    ${zapHome}/zap.sh -cmd \
        -quickurl ${remoteHost} \
        -quickprogress \
        -quickout ${reportNameHtml}
""", returnStdout: true).trim()
echo "ZAP Output: ${zapOutput}"
            
            // Archive the ZAP reports
           // archiveArtifacts artifacts: "${reportNameHtml},${reportNameXml}", fingerprint: true
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
            // Create a virtual environment and install requests using bash
            sh '''
                python3 -m venv venv
                bash -c "source venv/bin/activate && pip install requests"
            '''
            
            // Function to upload reports to DefectDojo
            def uploadToDefectDojo = {
    def scriptContent = """
import requests
import json
import sys
import os

def upload_report(report_path, report_type):
    url = "${DEFECTDOJO_URL}/api/v2/import-scan/"
    headers = {
        'Authorization': 'Token ${DEFECTDOJO_API_KEY}',
        'Accept': 'application/json'
    }
    data = {
        'product_name': '${PRODUCT_NAME}',
        'engagement': '${ENGAGEMENT_ID}',
        'scan_type': report_type,
        'active': 'true',
        'verified': 'true',
    }
    
    print(f"--- Attempting to upload {report_type} report ---")
    print(f"Report path: {report_path}")
    print(f"URL: {url}")
    print(f"Headers: {json.dumps({k: v if k != 'Authorization' else '[REDACTED]' for k, v in headers.items()}, indent=2)}")
    print(f"Data: {json.dumps(data, indent=2)}")
    
    try:
        if not os.path.exists(report_path):
            print(f"Error: Report file {report_path} does not exist")
            return False
        
        file_size = os.path.getsize(report_path)
        print(f"File size: {file_size} bytes")
        
        with open(report_path, 'rb') as file:
            files = {'file': file}
            response = requests.post(url, headers=headers, data=data, files=files)
        
        print(f"Response status code: {response.status_code}")
        print(f"Response content: {response.text}")
        
        if response.status_code == 201:
            print(f"Successfully uploaded {report_type} report")
            return True
        else:
            print(f"Failed to upload {report_type} report. Status code: {response.status_code}")
            return False
    except Exception as e:
        print(f"Error occurred while uploading {report_type} report: {str(e)}")
        return False

# Attempt to upload each report
reports = [
    ('gitleaks-report.json', 'Gitleaks Scan'),
    ('report/dependency-check-report.xml', 'Dependency Check Scan'),
    ('nikto_output.json', 'Nikto Scan')
   
    
]


success_count = 0
for report_path, report_type in reports:
    if upload_report(report_path, report_type):
        success_count += 1
    else:
        print(f"Failed to upload {report_type} report")

print(f"Summary: Successfully uploaded {success_count} out of {len(reports)} reports")

if success_count < len(reports):
    sys.exit(1)  # Exit with error if not all reports were uploaded
"""
    writeFile file: 'upload_to_defectdojo.py', text: scriptContent
    // Run the Python script in the virtual environment using bash
    return sh(script: 'bash -c "source venv/bin/activate && python3 upload_to_defectdojo.py"', returnStatus: true)
}
            def uploadStatus = uploadToDefectDojo()
            
            if (uploadStatus != 0) {
                unstable('Some reports failed to upload to DefectDojo')
            }
        }
    }
}



   
}
}
