pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/ranjitrupnawar/zomato-clone.git'
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv(installationName: 'sonar-scanner', credentialsId: 'sonar-token') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=zomato-clone -Dsonar.projectName=zomato-clone \
                          -Dsonar.login=$SONAR_AUTH_TOKEN'''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {   
                        sh 'docker build -t zomato .'
                        sh 'docker tag zomato ranjitrupnawr/zomato:latest'
                        sh 'docker push ranjitrupnawr/zomato:latest'
                    }
                }
            }
        }
        stage('TRIVY') {
            steps {
                sh 'trivy image ranjitrupnawr/zomato:latest > trivy.txt'
            }
        }
        stage('Deploy to Container') {
            steps {
                sh 'docker run -d --name zomato -p 3000:3000 ranjitrupnawr/zomato:latest'
            }
        }
    }
}

