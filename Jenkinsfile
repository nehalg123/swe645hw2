pipeline {
    agent any

    environment {
        KUBE_NAMESPACE = 'hw2'
        KUBE_DEPLOYMENT_NAME = 'surveyform-deployment'
        DOCKER_UNAME = ''
        DOCKER_PWD = ''
        BUILD_TIMESTAMP = ''
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/nehalg123/645hw2.git'
                }
            }
        }

        stage('Set Docker Credentials') {
            steps {
                script {
                    // Set Docker credentials for the entire pipeline
                    BUILD_TIMESTAMP = new Date().format("yyyyMMddHHmmss")
                    echo "Timestamp: ${BUILD_TIMESTAMP}"
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_UNAME', passwordVariable: 'DOCKER_PWD')]) {
                        DOCKER_UNAME = env.DOCKER_UNAME
                        DOCKER_PWD = env.DOCKER_PWD
                    }
                }
            }
        }

        stage('Build WAR') {
            steps {
                script {
                    //Build the WAR file
                    sh 'rm -rf *.war'
                    sh 'jar -cvf surveyform.war src/index.html'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    //withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_UNAME', passwordVariable: 'DOCKER_PWD')]) {
                    // Build Docker image
                        sh "echo ${BUILD_TIMESTAMP}"
                        sh "echo ${DOCKER_PWD} | docker login -u ${DOCKER_UNAME} --password-stdin"
                        sh "docker build -t ${DOCKER_UNAME}/survey-app:${BUILD_TIMESTAMP} . --platform linux/amd64"
                    //}
                }
            }
        }

        stage('Push Docker Image to Registry') {
            steps {
                script {
                    // Push Docker image to your Docker registry
                    sh "docker push ${DOCKER_UNAME}/survey-app:${BUILD_TIMESTAMP}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh "sed -i 's|image: .*|image: ${DOCKER_UNAME}/survey-app:${BUILD_TIMESTAMP}|' kubernetes/deployment.yaml"
                    sh "sed -i 's|namespace: .*|namespace: ${KUBE_NAMESPACE}|' kubernetes/deployment.yaml"
                    sh "kubectl apply -f kubernetes/deployment.yaml"
                }
            }
        }
    }

    post {
        always {
            // Clean up Docker images locally after build
            cleanWs()
            script {
                sh "docker rmi ${DOCKER_UNAME}/survey-app:${BUILD_TIMESTAMP}"
            }
        }
    }
}
