pipeline {

    agent any

    tools {
        jdk 'JDK21'
        nodejs 'NodeJS24'
    }

    environment {
        SCANNER_HOME = tool 'SonarScanner'

        DOCKER_IMAGE = "nishchitavc/react-cicd-pipeline:latest"

        DOCKER_CREDENTIALS = "Dockerhub"
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/nishchitavc21/React-CICD-Pipeline.git'
            }
        }

        stage('Verify Tools') {
            steps {
                bat 'java -version'
                bat 'node -v'
                bat 'npm -v'
                bat 'docker --version'
                bat 'kubectl version --client'
                bat 'trivy --version'
            }
        }

        stage('Install Dependencies') {
            steps {
                bat 'npm install'
            }
        }

        stage('Build React Application') {
            steps {
                bat 'npm run build'
            }
        }

        stage('Run Tests') {
            steps {
               bat 'npm test -- --watchAll=false --passWithNoTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    bat """
                    "%SCANNER_HOME%\\bin\\sonar-scanner.bat"
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Trivy File System Scan') {
            steps {
                bat 'trivy fs .'
            }
        }

        stage('Docker Build') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE% .'
            }
        }

        stage('Trivy Docker Image Scan') {
            steps {
                bat 'trivy image %DOCKER_IMAGE%'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'Dockerhub',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                    echo | set /p=\"%DOCKER_PASS%\" | docker login -u %DOCKER_USER% --password-stdin
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat 'docker push %DOCKER_IMAGE%'
            }
        }

        stage('Terraform Init') {
            steps {
                dir('terraform') {
                    bat 'terraform init'
                }
            }
        }

        stage('Terraform Validate') {
            steps {
                dir('terraform') {
                    bat 'terraform validate'
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                dir('terraform') {
                    bat 'terraform plan'
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                dir('terraform') {
                    bat 'terraform apply -auto-approve'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat '''
                set KUBECONFIG=C:\\Jenkins\\config
                kubectl config current-context
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                bat '''
                set KUBECONFIG=C:\\Jenkins\\config
                kubectl get pods
                kubectl get services
                kubectl rollout status deployment/react-cicd-deployment
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }

        success {
            echo 'Application deployed successfully.'
        }

        failure {
            echo 'Pipeline failed. Please check the console output.'
        }
    }
}