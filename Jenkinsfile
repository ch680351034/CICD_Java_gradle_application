pipeline{
    agent any
    stages{
        stage("sonar static analysis"){
            agent {
                    docker { image 'openjdk:11' }
                }
            steps{
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-jenkins') {
                        sh '''
                           chmod +x gradlew
                           ./gradlew sonarqube
                           '''
                    }

                    timeout(time: 1, unit: 'HOURS') {
              def qg = waitForQualityGate()
              if (qg.status != 'OK') {
                  error "Pipeline aborted due to quality gate failure: ${qg.status}"
                }
            }

                }

            


            }
           
        }
        
  
        
    }
    post{
        always{
            echo "==always=="
        }
        success{
            echo "===pipeline executed successfully ==="
        }
        failure{
            echo "===pipeline execution failed=="
        }
    }
}
