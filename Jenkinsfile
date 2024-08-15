pipeline {
    agent any 
    tools {
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

       // stage('Check Git Secrets With Gitleaks') {
            //steps {
                // Remove any existing report file
               // sh 'rm gitleaks_report.json || true'
                
                // Pull the Gitleaks Docker image
               // sh 'sudo docker pull zricethezav/gitleaks'
                
                // Run Gitleaks in a Docker container
               // sh 'sudo docker run --rm -v /var/lib/jenkins/workspace/vul-django:/repo zricethezav/gitleaks detect --source /repo --report-path /repo/gitleaks_report.json --report-format json'
                
                // Display the Gitleaks report
               // sh 'cat gitleaks_report.json'
            //}
        //}
        
        stage('Deploy with Docker Compose') {
            steps {
                script {
                    // Pull the base image and start services
                    sh 'docker-compose pull'
                    sh 'docker-compose up -d --build'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Since we're building with docker-compose, we'll need to push the image manually
                    docker.withRegistry("${REGISTRY}", DOCKER_CREDENTIALS_ID) {
                        sh "docker-compose push"
                    }
                }
            }
        }

        stage('Deploy to GCP VM') {
            steps {
                sshagent(['tomcatkey']) {
                    //sh '''
                    //ssh -o StrictHostKeyChecking=no abuabdillah5444@34.30.50.129 "docker-compose pull && docker-compose up -d"
                    //'''
                    sh '''
                    scp -o StrictHostKeyChecking=no docker-compose.yml abuabdillah5444@34.30.50.129:/home/abuabdillah5444/
                    ssh -o StrictHostKeyChecking=no abuabdillah5444@34.30.50.129 "cd /home/abuabdillah5444 && docker-compose pull && docker-compose up -d"
                    '''
                }
            }
        }
    }
    
    post {
        always {
            // Clean up after the build
            deleteDir()
        }
    }
}
