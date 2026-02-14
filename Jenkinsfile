pipeline {
    agent any

    environment {
        DOCKER_IMAGE     = "rakeshramch/static-web"
        DOCKER_TAG       = "${BUILD_NUMBER}"
        KUBE_NAMESPACE   = "default"
        DEPLOYMENT_NAME  = "static-web-deployment"
        CONTAINER_NAME   = "static-web"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/rakeshramch85-eng/static-wabsite_latest.git'
            }
        }

        stage('Docker Build') {
            steps {
                bat """
                docker version || exit 1
                docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% .
                """
            }
        }

        stage('Docker Hub Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                    docker logout
                    docker login -u %DOCKER_USER% -p %DOCKER_PASS%
                    """
                }
            }
        }

        stage('Docker Push') {
            steps {
                bat """
                docker push %DOCKER_IMAGE%:%DOCKER_TAG%
                """
            }
        }

        stage('Kubernetes Test') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    bat """
                    echo Using kubeconfig: %KUBECONFIG%
                    kubectl version --client
                    kubectl get nodes
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    bat """
                    kubectl apply -f deployment.yaml -n %KUBE_NAMESPACE%
                    kubectl apply -f service.yaml -n %KUBE_NAMESPACE%

                    kubectl set image deployment/%DEPLOYMENT_NAME% %CONTAINER_NAME%=%DOCKER_IMAGE%:%DOCKER_TAG% -n %KUBE_NAMESPACE%

                    kubectl rollout status deployment/%DEPLOYMENT_NAME% -n %KUBE_NAMESPACE%
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD SUCCESS – Docker image pushed & Kubernetes deployed"
        }
        failure {
            echo "❌ CI/CD FAILED – Check Jenkins console logs"
        }
    }
}
