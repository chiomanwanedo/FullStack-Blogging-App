pipeline {
    agent any

    tools {
        maven 'Maven 3'         // Must match the name configured in Jenkins Global Tool Configuration
        jdk 'jdk17'           // Optional: Only if your project needs Java 17
    }

    environment {
        DOCKER_IMAGE = "chiomavee/blogging-app:${BUILD_NUMBER}"  // Customize this with your DockerHub username/repo
        SONARQUBE_SERVER = 'SonarQube'                                       // Name of the SonarQube instance configured in Jenkins
        NEXUS_URL = 'http://192.168.0.150/8081/repository/maven-releases/'          // Nexus Maven repo URL
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'github', url: 'https://github.com/chiomanwanedo/FullStack-Blogging-App.git'
            }
        }

        stage('Maven Build & Unit Test') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy fs . --exit-code 0 || true'  // Allow build to continue even with vulnerabilities
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh 'mvn deploy -DaltDeploymentRepository=nexus::default::http://192.168.0.150:8081/repository/maven-releases/'
                }
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
                sh '''
                    kubectl set image deployment/blogging-app blogging-app=$DOCKER_IMAGE
                '''
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
