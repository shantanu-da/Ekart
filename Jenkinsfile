pipeline {
    agent any

    tools {
        // Set the default JDK to 17 for the entire pipeline
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
                // Compile stage uses the default JDK (jdk17)
                sh "mvn clean compile -DskipTests=true"
            }
        }

        stage('OWASP Scan') {
            steps {
                // Dependency check stage uses the default JDK (jdk17)
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Static Code Analysis') {
            environment {
                SONAR_URL = "http://52.63.226.147:9000"
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
                // Build stage uses the default JDK (jdk17)
                sh "mvn clean package -DskipTests=true"
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    // Docker stage uses the default JDK (jdk17)
                        withCredentials([usernamePassword(credentialsId: 'Dockerhub_login', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                        def buildNumber = env.BUILD_NUMBER ?: 'latest'
                        sh "docker build -t shan123456/docker_demo -f docker/Dockerfile ."
                        sh "docker tag shan123456/docker_demo:latest shan123456/docker_demo:${buildNumber}"
                        sh "docker login -u shan6101995@gmail.com -p shantanuu"
                        sh "docker push shan123456/docker_demo:${buildNumber}"
                        sh "docker push shan123456/docker_demo:latest"
                    }
                }
            }
        }
    }
}
