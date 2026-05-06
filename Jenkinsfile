pipeline {
    agent any 

    environment {
        DOCKER_CREDENTIALS_ID = 'roseaw-dockerhub'  
        DOCKER_IMAGE = 'cithit/khanalb'
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        GITHUB_URL = 'https://github.com/khanalb58/cybersafe-app.git'
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: "${GITHUB_URL}"]]])
            }
        }

        stage('Static Code Testing') {
            steps {
                script {
                    echo "Running Security Audit..."
                    sh 'npm install'
                    sh 'npm audit --audit-level=high'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry-1.docker.io', "${DOCKER_CREDENTIALS_ID}") {
                        docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push('latest')
                    }
                }
            }
        }

        stage('Deploy to Dev Environment') {
            steps {
                withCredentials([file(credentialsId: 'khanalb-225', variable: 'KUBECONFIG')]) {
                    script {
                        echo "Deploying to Kubernetes Cluster..."
                        
                        // FIX: Updated to match your actual filename 'deployment.yaml'
                        sh "sed -i 's|cithit/khanalb:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment.yaml"
                        
                        // FIX: Using your exact filenames from the repo
                        sh "kubectl --kubeconfig=${KUBECONFIG} apply -f deployment.yaml"
                        sh "kubectl --kubeconfig=${KUBECONFIG} apply -f k8s-service.yaml"
                    }
                }
            }
        }
        
        stage('Check Kubernetes Cluster') {
            steps {
                withCredentials([file(credentialsId: 'khanalb-225', variable: 'KUBECONFIG')]) {
                    script {
                        sh "kubectl --kubeconfig=${KUBECONFIG} get all"
                    }
                }
            }
        }
    }

    post {
        success {
            slackSend color: "good", message: "Build Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
        failure {
            slackSend color: "danger", message: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
    }
}
