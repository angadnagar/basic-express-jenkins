pipeline{
    agent{
        docker{
            image 'angadnagar/node-docker-agent:latest'
            args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    stages{
        stage('checkout'){
            steps{
                git branch: 'main', url: 'https://github.com/angadnagar/basic-express-jenkins.git'

            }
        }

        stage('Debug'){
            steps{
                 sh '''
                  echo " Current directory: $(pwd)"
                  echo " Files:"
                  ls -la
                  echo " Checking git status:"
                  git status
                '''
            }

        }

        stage('Build Push Docker Image'){
            environment {
            DOCKER_IMAGE = "angadnagar/jenkins-cicd:${BUILD_NUMBER}"
            REGISTRY_CREDENTIALS = credentials('docker-cred')
            }

            steps{
                script {
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                    
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    
                    docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                        dockerImage.push()
                    }
                }

            }
        }

        

        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "basic-express-jenkins"
                GIT_USER_NAME = "angadnagar"
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "nagarangad03@gmail.com"
                    git config user.name "Angad Nagar"
                    BUILD_NUMBER=${BUILD_NUMBER}
                
                    # Update deployment.yml
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" k8s/deployment.yml

                    git add k8s/deployment.yml
                    git commit -m "Update deployment image tag to version ${BUILD_NUMBER}"
                
                    # Push to main branch using token
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME}.git HEAD:main
                    '''
                }
            }
        }


    }
}
