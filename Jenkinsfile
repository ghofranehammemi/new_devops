pipeline {
    agent any

    tools {
        maven 'Maven'
        git 'Default'
    }

    environment {
        MAVEN_HOME = "${tool 'Maven'}"
        PATH = "${env.MAVEN_HOME}/bin:${env.PATH}"

        DOCKER_IMAGE = "ghofranehammemi/student-management"
        DOCKER_TAG = "1.0.${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = "dockerhub-credentials"

        SONAR_PROJECT_KEY = "student-management"
        SONAR_HOST_URL = "http://localhost:9090"
        SONAR_TOKEN = "7faa9a738feeecb4b412ae19f6e32546625ae312"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ghofranehammemi/new_devops.git'
            }
        }

        stage('Compile') {
            steps {
                sh './mvnw clean compile'
            }
        }

        stage('Test') {
            steps {
                sh './mvnw test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                        ./mvnw sonar:sonar \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }

        stage('Package') {
            steps {
                sh './mvnw package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                    docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS_ID}",
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    sh """
                        echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker push ${DOCKER_IMAGE}:latest
                        docker logout
                    """
                }
            }
        }
    }

    post {
        always {
            sh """
                docker rmi ${DOCKER_IMAGE}:${DOCKER_TAG} || true
                docker rmi ${DOCKER_IMAGE}:latest || true
            """
            cleanWs()
        }
        success {
            echo 'Pipeline réussi!'
        }
        failure {
            echo 'Pipeline a échoué!'
        }
    }
}
