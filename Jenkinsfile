pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "akshayamanimuthu/cloudnative-trend"
        AWS_REGION = "ap-south-1"
        EKS_CLUSTER = "cloudnative-eks"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                      -t ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                      -t ${DOCKER_IMAGE}:dev \
                      .
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKERHUB_USERNAME',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$DOCKERHUB_TOKEN" | docker login \
                          -u "$DOCKERHUB_USERNAME" \
                          --password-stdin

                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:dev

                        docker logout
                    '''
                }
            }
        }

        stage('Configure EKS') {
            steps {
                sh '''
                    aws eks update-kubeconfig \
                      --region ${AWS_REGION} \
                      --name ${EKS_CLUSTER}
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    kubectl apply -f kubernetes/deployment.yaml
                    kubectl apply -f kubernetes/service.yaml

                    kubectl set image deployment/trendify \
                      trendify=${DOCKER_IMAGE}:${BUILD_NUMBER}

                    kubectl rollout status deployment/trendify
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get pods
                    kubectl get service trendify-service
                '''
            }
        }
    }

    post {
        success {
            echo 'Trendify deployment completed successfully.'
        }

        failure {
            echo 'Trendify deployment failed. Check the Jenkins console output.'
        }
    }
}