pipeline {
    agent any
    environment {
        
        DOCKER_IMAGE = "santhhoshkumar/python:${BUILD_NUMBER}" // Moved here for global access
        REGISTRY_CREDENTIALS = credentials('dockerhub')
    }
    
        stage("Code") {
            steps {
                git "https://github.com/santhoshsandy1920/argocd.git"
            }
        }
       
        
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image with tag: ${DOCKER_IMAGE}"
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    echo "Pushing Docker image to Docker Hub..."
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', "dockerhub") {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "argocd"
                GIT_USER_NAME = "santhoshsandy1920"
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        # Configure Git
                        git config user.email "praveenchoudarychinthala@gmail.com"
                        git config user.name "Praveenchoudary"
                        
                        # Update the deployment file with the new image tag
                        sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" deployment.yml
                        
                        # Stage and commit the modified file
                        git add deployment/deployment-svc.yml
                        if git diff --cached --quiet; then
                            echo "No changes to commit."
                        else
                            git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                            
                            # Push changes to the GitHub repository
                            git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git HEAD:master
                        fi
                    '''
                }
            }
        }
    }
    
}
