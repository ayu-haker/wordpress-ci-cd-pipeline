pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        NAMESPACE = 'wp-project'
        KUBECONFIG_CRED_ID = 'kubeconfig-cred'
        REPO_URL = 'https://github.com/ayu-haker/wordpress-ci-cd-pipeline.git'
        BRANCH = 'main'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${BRANCH}", url: "${REPO_URL}"
            }
        }

        stage('Validate Manifests') {
            steps {
                withCredentials([
                    file(credentialsId: "${KUBECONFIG_CRED_ID}", variable: 'KUBECONFIG')
                ]) {
                    sh '''
                        set -e

                        echo "Checking kubectl..."
                        kubectl version --client

                        echo "Checking Kubernetes cluster..."
                        kubectl get nodes

                        echo "Validating manifests..."

                        kubectl apply --dry-run=client -f namespace-secret.yaml
                        kubectl apply --dry-run=client -f mysql-deployment.yaml
                        kubectl apply --dry-run=client -f wordpress-deployment.yaml

                        echo "✅ Manifest validation successful"
                    '''
                }
            }
        }

        stage('Deploy Namespace & Secret') {
            steps {
                withCredentials([
                    file(credentialsId: "${KUBECONFIG_CRED_ID}", variable: 'KUBECONFIG')
                ]) {
                    sh '''
                        set -e

                        echo "Deploying namespace and secret..."

                        kubectl apply -f namespace-secret.yaml

                        echo "✅ Namespace and secret deployed"
                    '''
                }
            }
        }

        stage('Deploy MySQL') {
            steps {
                withCredentials([
                    file(credentialsId: "${KUBECONFIG_CRED_ID}", variable: 'KUBECONFIG')
                ]) {
                    sh '''
                        set -e

                        echo "Deploying MySQL..."

                        kubectl apply -f mysql-deployment.yaml

                        echo "Waiting for MySQL rollout..."

                        kubectl -n ${NAMESPACE} rollout status \
                            deployment/mysql \
                            --timeout=180s

                        echo "✅ MySQL deployed successfully"
                    '''
                }
            }
        }

        stage('Deploy WordPress') {
            steps {
                withCredentials([
                    file(credentialsId: "${KUBECONFIG_CRED_ID}", variable: 'KUBECONFIG')
                ]) {
                    sh '''
                        set -e

                        echo "Deploying WordPress..."

                        kubectl apply -f wordpress-deployment.yaml

                        echo "Waiting for WordPress rollout..."

                        kubectl -n ${NAMESPACE} rollout status \
                            deployment/wordpress \
                            --timeout=300s

                        echo "✅ WordPress deployed successfully"
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([
                    file(credentialsId: "${KUBECONFIG_CRED_ID}", variable: 'KUBECONFIG')
                ]) {
                    sh '''
                        set -e

                        echo "===== NODES ====="
                        kubectl get nodes

                        echo "===== PODS ====="
                        kubectl -n ${NAMESPACE} get pods -o wide

                        echo "===== SERVICES ====="
                        kubectl -n ${NAMESPACE} get svc

                        echo "===== DEPLOYMENTS ====="
                        kubectl -n ${NAMESPACE} get deployments

                        echo "===== ROLLOUT STATUS ====="

                        kubectl -n ${NAMESPACE} rollout status \
                            deployment/mysql \
                            --timeout=180s

                        kubectl -n ${NAMESPACE} rollout status \
                            deployment/wordpress \
                            --timeout=300s

                        echo "✅ Kubernetes deployment verified successfully"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "========================================="
            echo "✅ WordPress CI/CD Pipeline SUCCESS"
            echo "Namespace: ${NAMESPACE}"
            echo "========================================="
        }

        failure {
            echo "========================================="
            echo "❌ WordPress CI/CD Pipeline FAILED"
            echo "Check the stage logs above."
            echo "========================================="
        }

        always {
            cleanWs()
        }
    }
}
