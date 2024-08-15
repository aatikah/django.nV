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
        ssh -o StrictHostKeyChecking=no abuabdillah5444@34.30.50.129 "sudo docker pull aatikah/vul-djangoapp:v1 && sudo docker run -d -p 8000:8000 aatikah/vul-djangoapp:v1"
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
        failure {
            // Send notification or take other actions on failure
            echo "Pipeline failed due to detected secrets."
        }
    }
}
