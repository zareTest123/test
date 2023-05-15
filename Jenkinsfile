pipeline {
    agent any
    options {
        runAsUser('root')
    }
    stages {
        stage('Configure microK8s permissions') {
            steps {
                sh 'sudo usermod -a -G microk8s jenkins'
                sh 'sudo chown -R jenkins ~/.kube'
            }
        }
        stage('Check Namespace') {
            steps {
                script {
                    def namespace = 'wp'
                    def namespaceExists = sh(script: "microk8s.kubectl get namespace $namespace", returnStatus: true)
                    if (namespaceExists == 0) {
                        echo "Namespace '$namespace' already exists."
                    } else {
                        echo "Namespace '$namespace' does not exist. Creating..."
                        sh "microk8s.kubectl create namespace $namespace"
                    }
                }
            }
        }
        stage('Install WordPress') {
            steps {
                script {
                    def releaseName = 'final-project-wp-scalefocus'
                    def chartPath = 'charts/bitnami/wordpress'
                    def namespace = 'wp'
                    def releaseExists = sh(script: "microk8s.helm status $releaseName --namespace $namespace", returnStatus: true)
                    if (releaseExists == 0) {
                        echo "WordPress release '$releaseName' already exists."
                    } else {
                        echo "WordPress release '$releaseName' does not exist. Installing..."
                        sh "microk8s.helm install $releaseName $chartPath --namespace $namespace"
                    }
                }
            }
        }
    }
}
