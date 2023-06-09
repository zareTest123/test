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

        stage('Create StorageClass') {
            steps {
                script {
                    sh '''
                    echo '
                    apiVersion: storage.k8s.io/v1
                    kind: StorageClass
                    metadata:
                      name: microk8s-storage
                    provisioner: kubernetes.io/no-provisioner
                    volumeBindingMode: WaitForFirstConsumer
                    ' | microk8s kubectl apply -f -
                    '''
                }
            }
        }

        stage('Check and Install WordPress') {
            steps {
                script {
                    def releaseStatus = sh(script: 'microk8s helm status final-project-wp-scalefocus -n wp', returnStatus: true)
                    if (releaseStatus != 0) {
                        sh 'microk8s helm repo add bitnami https://charts.bitnami.com/bitnami'
                        sh 'microk8s kubectl apply -f storage-class.yaml'
                        sh 'microk8s helm upgrade --install final-project-wp-scalefocus bitnami/wordpress -n wp -f charts/bitnami/wordpress/values.yaml'
                        
                        sh '''
                        echo '
                        apiVersion: networking.k8s.io/v1
                        kind: Ingress
                        metadata:
                          name: wordpress-ingress
                          namespace: wp
                        spec:
                          rules:
                            - host: localhost
                              http:
                                paths:
                                  - pathType: Prefix
                                    path: /
                                    backend:
                                      service:
                                        name: final-project-wp-scalefocus
                                        port:
                                          number: 80
                        ' | microk8s.kubectl apply -f -
                        '''
                    }
                }
            }
        }
    }
}
