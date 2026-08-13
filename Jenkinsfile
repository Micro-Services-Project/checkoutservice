pipeline {
    agent any 
    environment {
        IMAGE_NAME = "obitomanu/checkoutservice:${GIT_COMMIT}"
    }
    stages {
        stage ("CleanWS"){
            steps {
                CleanWs()
            }
        }
        stage("Git-Checkout") {
            steps {
                git branch: 'main', url: 'https://github.com/Micro-Services-Project/checkoutservice.git'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'Sonar'

                    withSonarQubeEnv('Sonar') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=checkoutservice \
                            -Dsonar.projectName=checkoutservice \
                            -Dsonar.sources=.
                        """
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'Sonar'
            }
        }
        stage("Run Unit Tests") {
            steps {
                sh 'go test ./...'
            }
        }
        stage("Build") {
            steps {
                sh """
                   printenv
                   docker build -t ${IMAGE_NAME} .
                   """
            }
        }
        stage("Scan") {
            steps {
                sh """ 
                   trivy image ${IMAGE_NAME} >> checkoutservice-report.txt
                   """
                   }
        }
        stage ("push Image") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker') {
                        sh """
                           docker push ${IMAGE_NAME}
                           """
                    }
                }
            }
        }
    }
}


