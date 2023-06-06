pipeline{
    agent{
        label "build"
    }
    environment{
        DOCKERHUB_USERNAME = "kenappiah"
        APP_NAME = "gitops-argo-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}/" + "${APP_NAME}"
        REGISTRY_CREDS = 'dockerhub'
    }

     stages{
        stage ("Cleanup Workspace"){
            steps {
                cleanWs()
            }
        }
        stage ("checkout scm"){
             steps {
                script {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/kenchedda/ARGO-PYTHON-CICD.git'
                }
             } 
        }
        stage ("build"){
            steps{
                script{ 
                     docker_image = docker.build "${IMAGE_NAME}"
                }
            }
        }   
        stage("push docker"){
              steps {
                script {
                    docker.withRegistry('', REGISTRY_CREDS){
                        docker_image.push("$BUILD_NUMBER")
                        docker_image.push('latest')
                    }
                }
         
             }
        } 

        stage("delete docker images"){
            steps{
                script{
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }     
       

    stage("updating kubernetes deployment file"){
        steps{
            script{

                sh  """
                cat deployment.yaml
                sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yaml
                cat deployment.yaml
                """
            }
        }
    }

    stage("push deployment.yaml to git"){ 
         steps {
            script {
                sh """
                git config --global user.name "kenchedda"
                git config --global user.email "kenappiah@gmail.com"
                git add deployment.yaml
                git commit -m "updated the deployment file"
                """
                withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                sh "git push https://github.com/kenchedda/ARGO-PYTHON-CICD.git main"
}
            }
         }   
     }
     stage("trigger cd pipeline"){
        steps {
            script{
                sh sh "curl -v -k --user admin:11e8055125d8db873b438c9ba575208302 -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://18.223.102.36:8080/job/cd/buildWithParameters?token=gitops-token'"
            }    
     }
     }

}  
}                  