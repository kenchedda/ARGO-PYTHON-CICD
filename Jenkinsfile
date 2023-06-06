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

}  
}                  