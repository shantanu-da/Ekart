pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
        dockerTool 'docker'
    }

    environment {
        SCANNER_HOME = tool 'SonarQube Scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: '15fb69c3-3460-4d51-bd07-2b0545fa5151', poll: false, url: 'https://github.com/shantanudatarkar/Ekart.git'
            }
        }

        stage('COMPILE') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }

        //stage('OWASP Scan') {
         //   steps {
         //       dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP'
         //       dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
         //   }
       // }
        
        stage('Static Code Analysis') {
            environment {
                SONAR_URL = "http://54.252.172.0:9000/"
            }
            steps {
                withCredentials([string(credentialsId: 'sonarQube-token', variable: 'sonarQube')]) {
                    echo "SonarQube Token: ${sonarQube}"
                    sh 'mvn sonar:sonar -Dsonar.login=$sonarQube -Dsonar.host.url=${SONAR_URL}'
                }
            }
        }

        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                        withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                        def buildNumber = env.BUILD_NUMBER ?: 'latest'
                        sh "docker build -t shan123456/docker_demo -f docker/Dockerfile ."
                        sh "docker tag shan123456/docker_demo:latest shan123456/docker_demo:${buildNumber}"
                        sh "docker push shan123456/docker_demo:${buildNumber}"
                        sh "docker push shan123456/docker_demo:latest"
                    }
                }
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                script {
                    withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'opstree1', contextName: 'arn:aws:eks:ap-southeast-2:442182381299:cluster/opstree1', credentialsId: 'k8s-credetials', namespace: 'default', serverUrl: 'https://821FFEB51BA5C7923ABB0F8F82546BE9.yl4.ap-southeast-2.eks.amazonaws.com']]) {
                        kubeconfig(credentialsId: 'k8s-credetials', serverUrl: 'https://821FFEB51BA5C7923ABB0F8F82546BE9.yl4.ap-southeast-2.eks.amazonaws.com') {
                            withCredentials([string(credentialsId: 'k8s-credetials', variable: 'k8s')]) {
                                sh 'cat $HOME/.kube/config'
                                sh 'kubectl config view'
                                sh "kubectl apply -f kubernetes/deploymentservice.yaml"
                            }
                        }
                    }
                }
            }
        }        
    }
}
