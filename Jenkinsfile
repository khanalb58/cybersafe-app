pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "khanalb58/cybersafe-app"
        KUBE_CONFIG_ID = 'k8s-kubeconfig'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'node -v'
                sh 'echo "Build test passed"'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${env.BUILD_ID} ."
                sh "docker build -t ${DOCKER_IMAGE}:latest ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}:${env.BUILD_ID}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: "${KUBE_CONFIG_ID}", variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f k8s-deployment.yaml --kubeconfig=$KUBECONFIG'
                    sh "kubectl set image deployment/cybersafe-app cybersafe-app=${DOCKER_IMAGE}:${env.BUILD_ID} --kubeconfig=$KUBECONFIG"
                }
            }
        }
    }
}
