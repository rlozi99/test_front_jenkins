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

        GIT_CREDENTIALS_ID = 'jenkins-git-access'

        KUBECONFIG = '/home/azureuser/.kube/config' // Update this path to where your kubeconfig is stored on Jenkins.

    }

    stages{
        stage('Check BRANCH_NAME') {
            steps {
                script {
                    echo "Current BRANCH_NAME is ${env.BRANCH_NAME}"
                }
            }
        }
        stage('Initialize..') {
            steps {
                script {
                    // Multibranch Pipeline에서 제공하는 BRANCH_NAME 환경 변수를 사용합니다.
                    def branch = env.BRANCH_NAME
                    echo "Checked out branch: ${branch}"
                    
                    if (branch == 'dev') {
                        env.TAG = 'dev'
                        env.DIR_NAME = "development"
                    } else if (branch == 'stg') {
                        env.TAG = 'stg'
                        env.DIR_NAME = "staging"
                    } else if (branch == 'prod') {
                        env.TAG = 'latest'
                        env.DIR_NAME = "droduction"
                    } else {
                        env.TAG = 'unknown'
                        env.DIR_NAME = "unknown"
                    }
                    
                    echo "TAG is now set to ${env.TAG}"
                }
            }
        }



        stage('Build and Push Docker Image to ACR') {
            steps {
                script {
                    sh "ls -la"
                    withCredentials([usernamePassword(credentialsId: 'acr-credential-id', passwordVariable: 'ACR_PASSWORD', usernameVariable: 'ACR_USERNAME')]) {
                        sh "az acr login --name $CONTAINER_REGISTRY --username $ACR_USERNAME --password $ACR_PASSWORD"
                        sh "docker build -t $CONTAINER_REGISTRY/$REPO:${TAG}${env.BUILD_ID} ."
                        sh "docker push $CONTAINER_REGISTRY/$REPO:${TAG}${env.BUILD_ID}"
                    }
                }
            }
        }

        stage('Checkout GitOps Repository') {
            steps {
                git branch: BRANCH_NAME, 
                credentialsId: GIT_CREDENTIALS_ID, 
                url: 'https://github.com/rlozi99/test-front-ops.git'
                script {
                    sh "ls -la"
                }
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

                        dir('overlays/${env.DIR_NAME}') {
                            sh "ls -la"
                            sh "kustomize build . | kubectl apply -f - -n eunjitest"
                        }
                    }
                }
            }
        }
        stage('Push Changes to GitOps Repository') {
            steps {
                script {
                    // 변경된 파일이 있는지 확인
                    def hasChanges = sh(script: "git status --porcelain", returnStdout: true).trim()
                    if (hasChanges) {
                        sh "git add ."
                        sh "git commit -m 'Update k8s configuration for ${env.BRANCH_NAME}'"
                        // sh "git push origin ${env.BRANCH_NAME}"

                        sh "git pull --rebase origin ${env.BRANCH_NAME}"
                        def remote = "https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/rlozi99/test_front_jenkins.git ${env.BRANCH_NAME}"
                        // 원격 저장소에 푸시
                        sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/rlozi99/test-front-ops.git ${env.BRANCH_NAME}"
                    } else {
                        echo "No changes to commit."
                    }
                }
            }
        }



        // stage('Push Changes to GitOps Repository') {
        //     steps {
        //         script {

        //             sh "git add ."
        //             sh "git commit -m 'Update k8s configuration for $BRANCH_NAME'"
        //             sh "git push origin $BRANCH_NAME"
        //             // withCredentials([usernamePassword(credentialsId: 'jenkins-git-access', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
        //             //     // 현재 브랜치 확인 및 main으로 체크아웃
        //             //     def currentBranch = sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
        //             //     if (currentBranch != "dev") {
        //             //         sh "git checkout dev"
        //             //     }
        //             //     // 원격 저장소에서 최신 변경사항 가져오기
        //             //     sh "git pull --rebase origin dev"
        //             //     def remote = "https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/rlozi99/test_front_jenkins.git"
        //             //     // 원격 저장소에 푸시
        //             //     sh "git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/rlozi99/test-front-ops.git main"
        //         }
        //     }
        // }
    }
}

// This assumes you have a 'withKubeConfig' shared library or function in Jenkins to handle kubeconfig.
def withKubeConfig(Map args, Closure body) {
    withCredentials([file(credentialsId: args.credentialsId, variable: 'KUBECONFIG')]) {
        body.call()
    }
}
    