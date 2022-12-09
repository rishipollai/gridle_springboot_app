pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
            steps{
                script{
                  withSonarQubeEnv(credentialsId: 'sonar-token') {
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
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]){
                    sh '''
                         docker build -t 34.125.242.216:8083/springapp:${VERSION} .
                         docker login -u admin -p $docker_password 34.125.242.216:8083 
                         docker push  34.125.242.216:8083/springapp:${VERSION}
                         docker rmi 34.125.242.216:8083/springapp:${VERSION}
                    '''
                    }
                }
            }
        }
        stage("identifying misconfuguration using datree in helm chart"){
            steps{
                script{
                    dir('kubernetes/') {
                        withEnv(['DATREE_TOKEN=GJdx2cP2TCDyUY3EhQKgTc']) {
                              sh 'helm datree test myapp/'
                        }
                    }
                }
            }

        }
    }
   
}
