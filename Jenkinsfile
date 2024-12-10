pipeline {
    agent any
    tools {
        jdk 'jdk11'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/jesuslk/springboot-demo.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Test') {
            steps {
                sh "mvn test"
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs.html . "
            }
        }
        stage('Build') {
            steps {
                sh "mvn package"
            }
        }
        stage("SonarQube Analysis"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh '''$SCANNER_HOME/bin/sonar-scanner \
                                -Dsonar.projectName=springboot \
                                -Dsonar.projectKey=springboot \
                                -Dsonar.java.binaries=target'''
                    }
                }
            }
        }
        stage('Docker build and tag') {
            steps {
                sh "docker build -t arce18/springboot1:latest ."
            }
        }
        stage('Trivy image Scan') {
            steps {
                sh "trivy image arce18/springboot1:latest --format table -o image.html"
            }
        }
        stage('Docker Push Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'dockerhub') {
                        sh "docker push arce1/springboot1:latest"
                  }
                }
            }
        }
		stage('Deploy to kubernets'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: 'EKS_CLOUD', contextName: '', credentialsId: 'k8s', namespace: 'default', restrictKubeConfigAccess: false, serverUrl: 'https://B18E12BF81A02624B83DD385816C9EF6.gr7.ap-south-1.eks.amazonaws.com') {
                        sh "kubectl apply -f deployment.yaml"
                        sh "kubectl apply -f service.yaml"
                    }
                }
            }
        }

    }
}
        
