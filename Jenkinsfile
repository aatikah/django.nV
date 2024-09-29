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
                    
                    
                    sh 'echo "Hello from Node"'
                    
    
                    
    
                   
               }
            }
            }
   

}
}
