pipeline {
    agent any

    tools {
        // Set the default JDK to 17 for the entire pipeline
        jdk 'jdk17'
        maven 'maven3'
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

        stage('Sonarqube') {
            steps {
                script {
                    echo "Jenkins Home: ${JENKINS_HOME}"
                    echo "SonarScanner Installation: ${tool 'SonarQube Scanner'}"
                    
                    // Switch to JDK 11 for SonarQube analysis
                    withEnv(["JAVA_HOME=${tool 'jdk11'}", "PATH=${tool 'jdk11'}/bin:${env.PATH}"]) {
                        withCredentials([string(credentialsId: 'sonarQube-token', variable: 'sonarToken')]) {
                            withSonarQubeEnv('SonarQube Scanner') {
                                sh """${tool 'SonarQube Scanner'}/bin/sonar-scanner \
                                    -Dsonar.projectName=Shopping-Cart \
                                    -Dsonar.java.binaries=. \
                                    -Dsonar.projectKey=Shopping-Cart \
                                    -Dsonar.login=${sonarToken}"""
                            }
                        }
                    }
                    // Add more debugging information if needed
                    sh "ls -la ${WORKSPACE}"
                    sh "cat ${WORKSPACE}/sonar-scanner-*.log"
                }
            }
        }

        // Continue with other stages...

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
                    withDockerRegistry(credentialsId: '2fe19d8a-3d12-4b82-ba20-9d22e6bf1672', toolName: 'docker') {
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag  shopping-cart adijaiswal/shopping-cart:latest"
                        sh "docker push adijaiswal/shopping-cart:latest"
                    }
                }
            }
        }
    }
}
