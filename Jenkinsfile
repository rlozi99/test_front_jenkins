pipeline {
    agent any

    environment {
        AZURE_SUBSCRIPTION_ID = 'c8ce3edc-0522-48a3-b7e4-afe8e3d731d9'
        AZURE_TENANT_ID = '4ccd6048-181f-43a0-ba5a-7f48e8a4fa35'
        CONTAINER_REGISTRY = 'goodbirdacr.azurecr.io'
        RESOURCE_GROUP = 'AKS'
        REPO = 'eunjitest_image/front'
        IMAGE_NAME = 'eunjitest_image/front:latest'
        NAMESPACE = 'eunjitest'
        TAG = 'latest'
        GIT_CREDENTIALS_ID = 'jenkins-git-access'
    }



    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Push Docker Image to ACR') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'acr-credential-id', passwordVariable: 'ACR_PASSWORD', usernameVariable: 'ACR_USERNAME')]) {
                        // Log in to ACR
                        sh "az acr login --name $CONTAINER_REGISTRY --username $ACR_USERNAME --password $ACR_PASSWORD"

                        // Build and push Docker image to ACR
                        // 변경: 이미지 이름을 $CONTAINER_REGISTRY/$IMAGE_NAME으로 수정
                        sh "docker build -t $CONTAINER_REGISTRY/$REPO:$TAG ."
                        sh "docker push $CONTAINER_REGISTRY/$REPO:$TAG"
                    }
                }
            }
        }
        stage('Checkout GitOps') {
                    steps {
                        // 'front_gitops' 저장소에서 파일들을 체크아웃합니다.
                        git branch: 'main',
                            credentialsId: 'jenkins-git-access',
                            url: 'https://github.com/rlozi99/test-front-ops'
                    }
                }
        stage('Update Kubernetes Configuration..') {
                    steps {
                        script {
                            sh "ls -la"
                            sh("""
                                kustomize build overlays/development | kubectl apply -f -
                            """)
                            // sh "git add ."
                            // sh "git commit -m 'Update image to ${TAG}'"
                        }
                    }
                }
        
    }
}