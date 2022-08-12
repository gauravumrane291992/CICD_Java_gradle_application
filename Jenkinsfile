pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-jenkins-token') {
                            sh 'chmod +x gradlew'
                            sh './gradlew sonarqube'
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
        stage("docker build & docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_Pass', variable: 'docker_Pass')]) {
                    sh '''
                    docker build -t 34.125.3.117:8083/springapp:${VERSION} .
                    docker login -u admin -p $docker_Pass 34.125.3.117:8083
                    docker push 34.125.3.117:8083/springapp:${VERSION}
                    docker rmi 34.125.3.117:8083/springapp:${VERSION}
                    '''
                    }                   
                }
            }
        }
        stage('indentify misconfigs using datree in helm charts'){
            steps{
                script{
                    dir('kubernetes/') {
                        withEnv(['DATREE_TOKEN=f6332b28-e5c3-4d8b-867e-557dbe09fb86']) {
                            sh 'helm datree test myapp/'
                        } 
                    }
                }
            }
        }
    }
}


