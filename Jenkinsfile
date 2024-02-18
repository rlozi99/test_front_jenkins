pipeline {
    agent any

    environment {
        AZURE_SUBSCRIPTION_ID = 'c8ce3edc-0522-48a3-b7e4-afe8e3d731d9'
        AZURE_TENANT_ID = '4ccd6048-181f-43a0-ba5a-7f48e8a4fa35'
        CONTAINER_REGISTRY = 'goodbirdacr.azurecr.io'
        RESOURCE_GROUP = 'AKS'
        REPO = 'eunjitest_image/front'
        IMAGE_NAME = 'eunjitest_image/front'
        NAMESPACE = 'eunjitest'
        TAG = 'dev'
        GIT_CREDENTIALS_ID = 'jenkins-git-access'
        KUBECONFIG = '/home/azureuser/.kube/config' // Update this path to where your kubeconfig is stored on Jenkins.
        BRANCH_NAME = 'dev' // 추가된 환경 변수
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
                script {
                    sh "ls -la"
                }
            }
        }

        stage('Build and Push Docker Image to ACR') {
            steps {
                script {
                    sh "ls -la"
                    withCredentials([usernamePassword(credentialsId: 'acr-credential-id', passwordVariable: 'ACR_PASSWORD', usernameVariable: 'ACR_USERNAME')]) {
                        sh "az acr login --name $CONTAINER_REGISTRY --username $ACR_USERNAME --password $ACR_PASSWORD"
                        sh "docker build -t $CONTAINER_REGISTRY/$REPO:$TAG ."
                        sh "docker push $CONTAINER_REGISTRY/$REPO:$TAG"
                    }
                }
            }
        }

        stage('Checkout GitOps Repository') {
            steps {
                git branch: BRANCH_NAME, credentialsId: GIT_CREDENTIALS_ID, url: 'https://github.com/rlozi99/test-front-ops.git'
            }
        }

        stage('Update Kubernetes Configuration') {
            steps {
                script {
                    sh "ls -la"
                    // Assuming the kubeconfig is set correctly on the Jenkins agent.
                    withKubeConfig([credentialsId: 'kubeconfig-credentials-id']) {
                        // Change directory to the location of your kustomization.yaml
                        sh "ls -la"
                        dir('overlays/development') {
                            sh "ls -la"
                            sh "kustomize build . | kubectl apply -f -"
                        }
                    }
                    withCredentials([usernamePassword(credentialsId: 'acr-credential-id', passwordVariable: 'ACR_PASSWORD', usernameVariable: 'ACR_USERNAME')]) {
                        sh "az acr login --name $CONTAINER_REGISTRY --username $ACR_USERNAME --password $ACR_PASSWORD"
                        sh "docker build -t $CONTAINER_REGISTRY/$REPO:$TAG ."
                        sh "docker push $CONTAINER_REGISTRY/$REPO:$TAG"
                    }
                }
            }
        }
    }
}

// This assumes you have a 'withKubeConfig' shared library or function in Jenkins to handle kubeconfig.
def withKubeConfig(Map args, Closure body) {
    withCredentials([file(credentialsId: args.credentialsId, variable: 'KUBECONFIG')]) {
        body.call()
    }
}
