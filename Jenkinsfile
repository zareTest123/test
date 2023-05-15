pipeline {
    agent any
    stages {
        stage('Check Namespace') {
            steps {
                sh 'usermod -a -G microk8s jenkins'
                sh 'chown -R jenkins ~/.kube'
                sh 'microk8s.kubectl get namespace wp'
            }
        }
        stage('Install WordPress') {
            steps {
                sh 'microk8s.helm install final-project-wp-scalefocus charts/bitnami/wordpress --namespace wp'
            }
        }
    }
}
