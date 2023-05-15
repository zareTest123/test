pipeline {
    agent any

    stages {
        stage('Check and Create Namespace') {
            steps {
                script {
                    sh 'sudo usermod -a -G microk8s jenkins'
                    sh 'sudo chown -R jenkins ~/.kube'
                    sh 'newgrp microk8s'
                    def namespaceExists = sh(script: 'microk8s kubectl get namespace wp', returnStatus: true)
                    if (namespaceExists != 0) {
                        sh 'microk8s kubectl create namespace wp'
                    }
                }
            }
        }
        stage('Check and Install WordPress') {
            steps {
                script {
                    def releaseStatus = sh(script: 'microk8s helm status final-project-wp-scalefocus -n wp', returnStatus: true)
                    if (releaseStatus != 0) {
                        sh 'microk8s helm repo add bitnami https://charts.bitnami.com/bitnami'
                        sh 'microk8s helm upgrade --install final-project-wp-scalefocus bitnami/wordpress -n wp -f values.yaml'
                    }
                }
            }
        }
    }
}
