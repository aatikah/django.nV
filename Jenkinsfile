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

        stage('Check Git Secrets With Trufflehog'){
            steps{
                sh 'rm trufflehogscan || true'
                sh 'sudo docker run gesellix/trufflehog --json https://github.com/aatikah/django.nV.git > trufflehogscan'
                sh 'cat trufflehogscan'
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
        ssh -o StrictHostKeyChecking=no abuabdillah5444@35.192.138.52 "sudo docker pull aatikah/vul-djangoapp:v1 && sudo docker run -d -p 8000:8000 aatikah/vul-djangoapp:v1"
        '''
    }
    }
}
        
        
    }
}
