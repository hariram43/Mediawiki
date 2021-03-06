pipeline {
    agent any
    
    parameters { 
         string(name: 'region', defaultValue: 'us-west-2', description: 'AWS Region')
         string(name: 'repo', defaultValue: '637373963597.dkr.ecr.us-west-2.amazonaws.com', description: 'ECR url')

    } 

    // triggers {
    //      pollSCM('* * * * *') // Polling Source Control
    //  }

stages{
        stage('Build'){              // build docker images
            steps {
                echo "Building images ... "

                sh 'docker build -t mediawiki ./docker/mediawiki/'
                sh 'docker build -t wikidb ./docker/database'

            }
        }
        stage ('Deploy to ECR'){     // deploy to aws registry
            steps {
                sh "\$(aws ecr get-login --no-include-email --region ${params.region})"
                sh "docker tag wikidb:latest ${params.repo}/wikidb:latest"
                sh "docker push ${params.repo}/wikidb:latest"
                sh "docker tag mediawiki:latest ${params.repo}/mediawiki:latest"
                sh "docker push ${params.repo}/mediawiki:latest"
            }
        }

        stage ('Deployments'){
            parallel{

                stage ("Deploy to Dev"){
                    steps {
                        sh "/usr/local/bin/kubectl apply -n dev -f ./k8s/mediawiki/deployment.yaml"    // Optinally we can give url of yaml file to apply
                        sh "/usr/local/bin/kubectl apply -n dev -f ./k8s/mediawiki/service.yaml"
                        sh "/usr/local/bin/kubectl apply -n dev -f ./k8s/wikidb/deployment.yaml"
                        sh "/usr/local/bin/kubectl -n dev -f ./k8s/wikidb/service.yaml"                        
                    }
                 }

                stage ("Deploy to Staging"){
                    steps {
                        script{
                            input "Do you want to proceed ?"
                            sh "kubectl apply -n stage -f ./k8s/mediawiki/deployment.yaml"
                            sh "kubectl apply -n stage -f ./k8s/mediawiki/service.yaml"
                            sh "kubectl apply -n stage -f ./k8s/wikidb/deployment.yaml"
                            sh "kubectl apply -n stage -f ./k8s/wikidb/service.yaml"
                        }
                    } 
                    }
                
                stage ("Deploy to Prod"){
                    steps {
                        script{
                            input "Do you want to proceed ?"
                            sh "kubectl apply -n prod -f ./k8s/mediawiki/deployment.yaml"
                            sh "kubectl apply -n prod -f ./k8s/mediawiki/service.yaml"
                            sh "kubectl apply -n prod -f ./k8s/wikidb/deployment.yaml"
                            sh "kubectl apply -n prod -f ./k8s/wikidb/service.yaml"
                        }
                    }
                }
            }
        }
    }
}
