pipeline{
    agent any
    environment {
        REGISTRY_NAME="k8sregistryy.azurecr.io"
        REPO_NAME="${REGISTRY_NAME}/springapp"
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

                   withCredentials([string(credentialsId: 'azure-conreg-password', variable: 'conregpasswd')]) {
                       sh '''
                       docker build -t ${REPO_NAME}:${TAG} .
                       docker login -u k8sregistryy -p ${conregpasswd} ${REGISTRY_NAME}
                       docker push ${REPO_NAME}:${TAG}
                       docker rmi ${REPO_NAME}:${TAG}

                       '''
                     }

                }
            }

        }

        stage("datree helm charts configuration Validation"){
            steps {
                dir('kubernetes') {

                   withEnv(['dat-token="o5sLwcMnBgzCS9qCvMtfTt"']) {
                       sh 'helm datree test myapp'
                   }

               }
            }
        }
        /* stage("publish the helm chart to repo "){

            steps{
                script {

                   withCredentials([string(credentialsId: 'docker-registry-pass', variable: 'dockerregpass')]) {
                       dir('kubernetes/') {
                             sh '''
                                 helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                 tar -czvf  myapp-${helmversion}.tar.gz myapp/
                                 curl -u admin:$dockerregpass http://212.2.243.207:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tar.gz -v
                            '''
                          }
                     }

                }
            }

        } */

        stage('Helm deployment') {
            steps {

                withCredentials([kubeconfigFile(credentialsId: 'azure-cluster-kconfig', variable: 'KUBECONFIG')]) {
                dir('kubernetes/'){

                   sh 'helm upgrade --install --set image.tag=${TAG} my-webapp myapp'
                }


                }

            }
        }

        stage('Manual approal') {
            steps {
                   mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> please click on follow link to approe/deny the deployment <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "chguru121@gmail.com";

                   input submitter: 'userId', message: 'Ready for deployment?'
                
                }


                }

            }
        }


    }
    post{
        always{

                                        mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "chguru121@gmail.com";

        }

    }
}

