pipeline {
    agent any

    tools {
        maven 'Maven 3'
        jdk 'jdk17'
    }

    environment {
        DOCKER_IMAGE = "chiomavee/blogging-app:${BUILD_NUMBER}"
        SONARQUBE_SERVER = 'sonarqube'
        NEXUS_URL = 'http://192.168.0.150:8081/repository/maven-releases/'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    credentialsId: 'github1',
                    url: 'https://github.com/chiomanwanedo/FullStack-Blogging-App.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy fs . --exit-code 0 || true'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                sh '''
                    mvn clean deploy \
                    -DaltDeploymentRepository=nexus::default::http://nexus:8081/repository/maven-releases/
                '''
            }
        }

        stage('Docker Build & Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker build -t $DOCKER_IMAGE .
                        docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl set image deployment/blogging-app blogging-app=$DOCKER_IMAGE"
                sh "kubectl rollout status deployment/blogging-app"
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
