pipeline {
    agent any

    environment {
        FRONTEND_IMAGE = "poojaelango/frontend-app:latest"
        BACKEND_IMAGE = "poojaelango/backend-app:latest"
        FRONTEND_CONTAINER = "frontend-container"
        BACKEND_CONTAINER = "backend-container"
        REGISTRY_CREDENTIALS = "docker"  // Jenkins credentials ID
    }

    stages {
        stage('Checkout Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'POOJZ', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    git url: "https://$GIT_USER:$GIT_TOKEN@github.com/POOJAELANGO03/DEV.git", branch: 'main'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    dir('frontend') {
                        sh 'docker build -t $FRONTEND_IMAGE .'
                    }
                    dir('backend') {
                        sh 'docker build -t $BACKEND_IMAGE .'
                    }
                }
            }
        }

        stage('Login to Docker Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                sh 'docker push $FRONTEND_IMAGE'
                sh 'docker push $BACKEND_IMAGE'
            }
        }

        stage('Stop & Remove Existing Containers') {
            steps {
                script {
                    sh '''
                    for container in $FRONTEND_CONTAINER $BACKEND_CONTAINER; do
                        if [ "$(docker ps -aq -f name=$container)" ]; then
                            docker stop $container || true
                            docker rm $container || true
                        fi
                    done
                    '''
                }
            }
        }

        stage('Run Docker Containers') {
            steps {
                sh 'docker run -d -p 3000:3000 --name $FRONTEND_CONTAINER $FRONTEND_IMAGE'
                sh 'docker run -d -p 5000:5000 --name $BACKEND_CONTAINER $BACKEND_IMAGE'
            }
        }
    }

    post {
        success {
            echo "Frontend and Backend: Build, push, and container execution successful!"
        }
        failure {
            echo "One or more stages failed."
        }
    }
}
