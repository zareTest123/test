pipeline {
    agent any

    stages {
        stage('Install microk8s') {
            steps {
                sh 'sudo snap install microk8s --classic'
                sh 'sudo usermod -a -G microk8s jenkins'
                sh 'sudo chown -f -R jenkins ~/.kube'
                sh 'sudo su - jenkins -c "microk8s config > ~/.kube/config"'
            }
        }
        stage('Check and Create Namespace') {
            steps {
                script {
                    def namespaceExists = sh(script: 'microk8s kubectl get namespace wp', returnStatus: true)
                    if (namespaceExists != 0) {
                        sh 'microk8s kubectl create namespace wp'
                    }
                }
            }
        }
        stage('Deploy WordPress') {
            steps {
                sh 'microk8s kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.46.0/deploy/static/provider/cloud/deploy.yaml'
                sh 'microk8s helm repo add bitnami https://charts.bitnami.com/bitnami'
                sh 'microk8s helm upgrade --install final-project-wp-scalefocus bitnami/wordpress -n wp -f values.yaml'
            }
        }
    }
}
