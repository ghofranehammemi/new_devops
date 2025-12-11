pipeline {
    agent any

    tools {
        maven 'Maven'
        jdk 'JDK-17'
    }

    options {
        timeout(time: 60, unit: 'MINUTES')
        retry(2)
    }

    environment {
        DOCKER_IMAGE = "ghofranehammemi/student-management"
        DOCKER_BUILDKIT = "1"
    }

    stages {

        /* --- CHECKOUT --- */
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/ghofranehammemi/new_devops.git'
            }
        }

        /* --- BUILD --- */
        stage('Build') {
            steps {
                sh 'mvn clean compile -B'
            }
        }

        /* --- TEST --- */
        stage('Test') {
            steps {
                sh 'mvn test -B'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        /* --- PACKAGE --- */
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests -B'
            }
        }

        /* --- SONARQUBE ANALYSIS --- */
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarServer') {
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_AUTH_TOKEN')]) {

                        sh """
                            mvn org.sonarsource.scanner.maven:sonar-maven-plugin:4.0.0.4121:sonar \
                            -Dsonar.projectKey=student-management \
                            -Dsonar.host.url=http://localhost:9000 \
                            -Dsonar.login=$SONAR_AUTH_TOKEN
                        """
                    }
                }
            }
        }

        /* --- DOCKER BUILD --- */
        stage('Docker Build') {
            steps {
                script {
                    retry(3) {
                        sh """
                            docker build --no-cache -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                            docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                        """
                    }
                }
            }
        }

        /* --- DOCKER PUSH --- */
        stage('Docker Push') {
            steps {
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'docker-hub-credentials',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )
                    ]) {

                        sh '''
                            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        '''

                        timeout(time: 20, unit: 'MINUTES') {
                            sh '''
                                docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                                docker push ${DOCKER_IMAGE}:latest
                            '''
                        }

                        sh 'docker logout'
                    }
                }
            }
        }
    }

    /* --- CLEANUP --- */
    post {
        always {
            sh """
                docker rmi ${DOCKER_IMAGE}:${BUILD_NUMBER} || true
                docker rmi ${DOCKER_IMAGE}:latest || true
            """
            cleanWs()
        }
        success {
            echo '✅ Pipeline réussi!'
        }
        failure {
            echo '❌ Pipeline échoué!'
        }
    }
}
