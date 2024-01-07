pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
        maven 'maven'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'ansible-sonar', url: 'https://github.com/rameshkumarvermagithub/sample-web-application.git'
            }
        }
       stage('Build code') {
            steps {
                sh "mvn clean package"
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=sample-web-application \
                    -Dsonar.projectKey=sample-web-application'''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar'
                }
            }
        }
        // stage('Install Dependencies') {
        //     steps {
        //         sh "npm install"
        //     }
        // }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
         stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh "docker build -t sample-web-application ."
                       sh "docker tag sample-web-application rameshkumarverma/sample-web-application:latest"
                       sh "docker push rameshkumarverma/sample-web-application:latest"
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image rameshkumarverma/sample-web-application:latest > trivyimage.txt"
            }
        }
        // stage("deploy_docker"){
        //     steps{
        //         sh "docker stop sample-web-application || true"  // Stop the container if it's running, ignore errors
        //         sh "docker rm sample-web-application || true" 
        //         sh "docker run -d --name sample-web-application -p 4000:3000 rameshkumarverma/sample-web-application:latest"
        //     }
        // }
      stage('Deploy to Kubernetes') {
            steps {
                script {
                    // dir('K8S') {
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                            // Apply deployment and service YAML files
                            sh 'kubectl apply -f deployment.yml'
                            sh 'kubectl apply -f service.yml'

                            // Get the external IP or hostname of the service
                            // def externalIP = sh(script: 'kubectl get svc amazon-service -o jsonpath="{.status.loadBalancer.ingress[0].hostname}"', returnStdout: true).trim()

                            // Print the URL in the Jenkins build log
                            // echo "Service URL: http://${externalIP}/"
                        }
                    // }
                }
            }
        }

    }
}
