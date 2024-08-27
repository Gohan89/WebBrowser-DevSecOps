pipeline {
    agent any
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage("Git Checkout") {
            steps {
                git branch: 'main', url: 'https://github.com/Gohan89/WebBrowser-DevSecOps.git'
            }
        }

        stage("OWASP Check") {
            steps {
                script {
                    dependencyCheck additionalArguments: '--scan ./ --disableNodeAudit', odcInstallation: 'owasp'
                }
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=WebBrowser \
                        -Dsonar.projectName=WebBrowser \
                        -Dsonar.sources=.
                    '''
                }
            }
        }

        stage("Docker Image Build") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-creds') {
                        dir('/home/ubuntu/.jenkins/workspace/WebBrowser/.docker/google-chrome') {
                            sh "docker build -t gohan789/webbrowser:latest ."
                        }
                    }
                }
            }
        }

        stage("Trivy Image Scan") {
            steps {
                sh "trivy image gohan789/webbrowser:latest"
            }
        }

        stage("Push Image to DockerHub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-creds') {
                        sh "docker push gohan789/webbrowser:latest"
                    }
                }
            }
        }

        stage("Deploy") {
            steps {
                sh "docker-compose up -d"
            }
        }
    }

}
