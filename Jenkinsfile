pipeline {
    agent {
        label 'slave-jenkins-deb'  // Replace with the label of your slave node
    }
    environment{
        DOCKER_REGISTRY = 'https://index.docker.io/v1/'
        DOCKER_IMAGE = 'aatikah/django-app'
        
    }
       //DOCKER_CREDENTIALS_ID = 'docker-credential'
    // REPOSITORY = 'aatikah'

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
            def remoteHost = '34.134.182.0'
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
            def zapHome = tool '/opt/zaproxy' // Path to ZAP installation
            def targetURL = 'http://34.134.182.0'  // Update this to your application's URL
            def reportNameHtml = 'zap-scan-report.html'
            def reportNameJson = 'zap-scan-report.json'
            
            // Perform ZAP scan
            sh """
                ${zapHome}/zap.sh -cmd \
                    -quickurl ${targetURL} \
                    -quickprogress \
                    -quickout ${reportNameHtml} \
                    -quickjson ${reportNameJson}
            """
            
            // Archive the ZAP reports
            archiveArtifacts artifacts: "${reportNameHtml},${reportNameJson}", fingerprint: true
            
            // Read and parse JSON report
            def zapJson = readJSON file: reportNameJson
            
            // Example: Check for high alerts in JSON
            def highAlerts = zapJson.site[0].alerts.findAll { it.riskcode >= 3 }
            
            if (highAlerts.size() > 0) {
                echo "Found ${highAlerts.size()} high-risk vulnerabilities!"
                highAlerts.each { alert ->
                    echo "High Risk Alert: ${alert.alert} at ${alert.url}"
                }
                error "OWASP ZAP scan found high-risk vulnerabilities. Check the ZAP report for details."
            }

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
            archiveArtifacts artifacts: 'zap-scan-report.json', fingerprint: true

            
        }
    }
   
           
     
}

   
}
}
