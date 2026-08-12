pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        NAMESPACE   = 'wp-project'
        KUBECONFIG_CRED_ID = 'kubeconfig-cred'   // Jenkins credential ID (Secret file) for your kubeconfig
        REPO_URL    = 'https://github.com/ayu-haker/wordpress-ci-cd-pipeline.git'
        BRANCH      = 'main'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: "${BRANCH}", url: "${REPO_URL}"
            }
        }

        stage('Validate Manifests') {
            steps {
                withCredentials([file(credentialsId: "${KUBECONFIG_CRED_ID}", variable: 'KUBECONFIG')]) {
                    sh '''
                        kubectl version --client
                        kubectl apply --dry-run=client -f namespace-secret.yaml
                        kubectl apply --dry-run=client -f mysql-deployment.yaml
                        kubectl apply --dry-run=client -f wordpress-deployment.yaml
                    '''
                }
            }
        }

        stage('Deploy Namespace & Secret') {
            steps {
                withCredentials([file(credentialsId: "${KUBECONFIG_CRED_ID}", variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f namespace-secret.yaml'
                }
            }
        }

        stage('Deploy MySQL') {
            steps {
                withCredentials([file(credentialsId: "${KUBECONFIG_CRED_ID}", variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f mysql-deployment.yaml'
                }
            }
        }

        stage('Deploy WordPress') {
            steps {
                withCredentials([file(credentialsId: "${KUBECONFIG_CRED_ID}", variable: 'KUBECONFIG')]) {
                    sh 'kubectl apply -f wordpress-deployment.yaml'
                }
            }
        }

        stage('Verify Rollout') {
            steps {
                withCredentials([file(credentialsId: "${KUBECONFIG_CRED_ID}", variable: 'KUBECONFIG')]) {
                    sh '''
                        kubectl -n ${NAMESPACE} rollout status deployment/mysql --timeout=120s || true
                        kubectl -n ${NAMESPACE} rollout status deployment/wordpress --timeout=180s
                        kubectl -n ${NAMESPACE} get pods,svc
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ WordPress CI/CD pipeline deployed successfully to namespace '${NAMESPACE}'."
        }
        failure {
            echo "❌ Pipeline failed. Check the stage logs above."
        }
        always {
            cleanWs()
        }
    }
}
