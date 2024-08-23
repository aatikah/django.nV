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
        DEFECTDOJO_API_KEY = credentials('DEFECTDOJO_API_KEY')
        DEFECTDOJO_URL = credentials('DEFECTDOJO_URL')
        ENGAGEMENT_ID = '2'
    
    }
    
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

        stage('Source Composition Analysis'){
            steps{
                sh 'rm owasp* || true'
                sh 'wget "https://raw.githubusercontent.com/aatikah/django.nV/master/owasp-dependency-check.sh"'
                sh 'chmod +x owasp-dependency-check.sh'
                sh 'bash owasp-dependency-check.sh'
                sh 'cat /var/lib/jenkins/workspace/vul-django/odc-reports/dependency-check-report.xml'
            }
        }

    

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

        stage('SAST With Sonarqube') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/master']],
                    userRemoteConfigs: [[url: 'https://github.com/aatikah/django.nV.git']]
                ])
                withSonarQubeEnv('vul-django') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh '''
                            export SONAR_TOKEN=${SONAR_TOKEN}
                            mvn sonar:sonar -Dsonar.token=$SONAR_TOKEN
                        '''
            }
        }
    }
}

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
                        script: '/home/abuabdillah5444/myenv/bin/bandit -r /var/lib/jenkins/workspace/vul-django -o bandit_report.json -f html',
                        returnStatus: true
                    )
        
                    if (banditHtmlStatus != 0) {
                        echo "Bandit found issues in the code. Please review the report at bandit_report.html."
                        // Optionally, you can fail the build if you want to treat these issues as critical
                        // error("Bandit scan failed with exit code ${banditStatus}")
                    }
                    
                    // Display the Bandit report
                    sh 'cat bandit_report.html'

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

                    def response = sh(
                        script: """
                        curl -X POST ${DEFECTDOJO_URL}/api/v2/import-scan/ \
                        -H "Authorization: Token ${DEFECTDOJO_API_KEY}" \
                        -H "accept: application/json" \
                        -H "Content-Type: multipart/form-data" \
                        -F "file=@bandit_report.json" \
                        -F 'scan_type=Bandit Scan' \
                        -F 'engagement=${ENGAGEMENT_ID}'
                        """,
                        returnStdout: true
                    ).trim()

                    echo "Response from DefectDojo: ${response}"

                    
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
        ssh -o StrictHostKeyChecking=no abuabdillah5444@34.31.246.184 "sudo docker pull aatikah/vul-djangoapp:v1 && sudo docker run -d -p 8000:8000 aatikah/vul-djangoapp:v1"
        '''
    }
    }
}
        
        
    }
    //post {
       // always {
            // Clean up after the build
           // deleteDir()
       // }
        
    //}
}
