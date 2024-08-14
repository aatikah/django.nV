pipeline {
    agent any 
   
    environment {
        REGISTRY = 'https://index.docker.io/v1/'
        REPOSITORY = 'aatikah'
        IMAGE_NAME = 'portfolio-website'
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
        
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${REPOSITORY}/${IMAGE_NAME}")
                }
            }
        }
        
        
    }
}
