pipeline{
    agent any
    environment {
        REGISTRY_NAME="212.2.243.207:8083"
        IMAGE_NAME="${REGISTRY_NAME}/springapp"
        TAG="${BUILD_ID}"
        
    }
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
        
        stage("code build and push "){
            
            steps{
                script {

                   withCredentials([string(credentialsId: 'docker-registry-pass', variable: 'dockerregpass')]) {
                       sh '''
                       docker build -t ${IMAGE_NAME}:${TAG} .
                       docker login -u admin -p ${dockerregpass} ${REGISTRY_NAME}
                       docker push ${IMAGE_NAME}:${TAG}
                       
                       '''
                     }

                    
                    
                 
                }

            }
           
        }
        

  
        
    }
    post{
        always{
            			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "chguru121@gmail.com";  
                      
        }
        success{
            echo "===pipeline executed successfully ==="
        }
        failure{
            echo "===pipeline execution failed=="
        }
    }
}
