    pipeline{
        agent any
        environment{
            version = "${env.BUILD_ID}"
        }
        stages{
            stage("sonar quality check"){
                /* agent {
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
            // and we are giving nexus ip every wheare
                steps{
                    script{
                       withCredentials([string(credentialsId: 'dockerpass', variable: 'dockerpass')]) {
                       sh 'docker build -t  34.125.30.90:8083/springboot:${version} .'
                       sh 'docker login -u admin -p $dockerpass 34.125.30.90:8083'
                       sh 'docker push  34.125.30.90:8083/springboot:${version}'
                       sh 'docker rmi  34.125.30.90:8083/springboot:${version}' 
                       
                       }
                    }
                }



        }
        stage("pushing images to neexus"){
            //here we are building docker image and pusing image 
                steps{
                       script{

                         withCredentials([string(credentialsId: 'nexuxpass', variable: 'nexux')]) {
                            dir('kubernetes/'){
                             sh '''   
                              helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                tar -czvf  myapp-${helmversion}.tgz myapp/
                              curl -u admin:$nexux http://34.125.30.90:8081/repository/helm-repo/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                            }
                         }
                        
                       
                        }
                    }
                }

            stage('Deploying application on k8s cluster') {
              steps {
                 script{
                         withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubeconfig', namespace: '', serverUrl: 'https://10.182.0.5:6443']])  {
                          dir('kubernetes/') {
                            sh 'helm upgrade --install --set image.repository="34.125.30.90:8083/springapp" --set image.tag="${version}" myjavaapp myapp/ '
                        }
                    }  
                }
             }
           }
           stage('verifying app deployment'){
                steps{
                    script{
                       withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubeconfig', namespace: '', serverUrl: 'https://10.182.0.5:6443']]) {
                         sh 'kubectl run curl --image=curlimages/curl -i --rm --restart=Never -- curl myjavaapp-myapp:8080'

                        }
                    }
                }
            }

        }
    }