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

        stage('SAST with Bandit') {
            steps {
                script {
                // Remove any existing Bandit report file
                sh 'rm -f bandit_report.html'
                    
                // Ensure Bandit is installed in the Jenkins environment
               // sh 'pip install bandit'
                
                // Run Bandit on the code directory
                sh 'bandit -r /var/lib/jenkins/workspace/vul-django -o bandit_report.html -f html'
                
                // Display the Bandit report
                sh 'cat bandit_report.html'
                // Archive the Bandit report as an artifact
                archiveArtifacts artifacts: 'bandit_report.html', allowEmptyArchive: true
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
        ssh -o StrictHostKeyChecking=no abuabdillah5444@34.173.148.142 "sudo docker pull aatikah/vul-djangoapp:v1 && sudo docker run -d -p 8000:8000 aatikah/vul-djangoapp:v1"
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
