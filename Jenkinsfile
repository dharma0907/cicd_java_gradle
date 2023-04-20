    pipeline{
        agent any
        environment{
            version = "${env.BUILD_ID}"
        }
        stages{
            stage("sonar quality check"){
                /*agent {
                    docker {
                        image 'openjdk:11'
                    }
                }*/
                steps{
                    script{
                     withSonarQubeEnv(credentialsId: 'sonarpassword') {
                       
                       sh 'chmod +x gradlew'
                       sh './gradlew sonarqube'
                       
                       }
                       timeout(5) {
                        def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                            }
                    }
                }   
        }
           stage("docker build and docker push"){
            //here we are building docker image and pusing image 
                steps{
                    script{
                       withCredentials([string(credentialsId: 'dockerpass', variable: 'dockerpass')]) {
                       sh 'docker build -t 34.125.215.209:8083/springboot:${version} .'
                       sh 'docker login -u admin -p $dockerpass 34.125.215.209:8083'
                       sh 'docker push 34.125.215.209:8083/springboot:${version}'
                       sh 'docker rmi 34.125.215.209:8083/springboot:${version}' 
                     }  

                    }
                }



           }
    }      
 }