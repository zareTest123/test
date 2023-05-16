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
                    microk8s kubectl apply -f - <<EOF
                    apiVersion: storage.k8s.io/v1
                    kind: StorageClass
                    metadata:
                      name: microk8s-storage
                    provisioner: kubernetes.io/no-provisioner
                    volumeBindingMode: WaitForFirstConsumer
                    EOF
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
                        sh 'microk8s helm upgrade --install final-project-wp-scalefocus bitnami/wordpress -n wp -f charts/bitnami/wordpress/values.yaml'
                        
                        sh '''
                        cat <<EOF | microk8s kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: final-project-wp-scalefocus
  namespace: wp
spec:
  type: LoadBalancer
  ports:
    - name: http
      port: 80
      targetPort: 8080
    - name: https
      port: 443
      targetPort: 8443
  selector:
    app.kubernetes.io/name: final-project-wp-scalefocus
EOF
                        '''
                    }
                }
            }
        }
    }
}
