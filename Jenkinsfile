pipeline {
    agent any 
   
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
                        dockerImage.push('latest')
                    }
                }
            }
        }
        stage('Deploy to GCP VM') {
        steps {
        sshagent(['tomcatkey']) {
        sh '''
        ssh -o StrictHostKeyChecking=no abuabdillah5444@35.192.138.52 "sudo docker pull aatikah/vul-djangoapp:latest && sudo docker run -d -p 8000:8000 aatikah/vul-djangoapp:latest"
        '''
    }
    }
}
        
        
    }
}
