pipeline {
    agent {
        label 'slave-jenkins-deb'  // Replace with the label of your slave node
    }
   tools{
       maven 'Maven'
   }
stages{
    stage('Initialize') {
                steps {
                    sh '''
                        echo "PATH = ${PATH}"
                        echo "M2_HOME = ${M2_HOME}"
                    ''' 
                }
            } 
    stage('Echo') {
            steps {
                script {
                    
                    //Remove any existing report file
                    sh 'echo "Hello from Node"'
                    
    
                    
    
                   
               }
            }
            }

}
}
