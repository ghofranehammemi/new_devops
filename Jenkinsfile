pipeline {
    agent any
    
    tools {
        maven 'Maven'
        jdk 'JDK'
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
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/ghofranehammemi/new_devops.git'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile -B'
            }
        }
        
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
        
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests -B'
            }
        }
        
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
        
        stage('Docker Push') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        retry(3) {
                            sh '''
                                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                                
                                # Push avec timeout et retry
                                timeout 600 docker push ${DOCKER_IMAGE}:${BUILD_NUMBER} || {
                                    echo "Push failed, retrying..."
                                    sleep 10
                                    docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                                }
                                
                                timeout 600 docker push ${DOCKER_IMAGE}:latest || {
                                    echo "Push failed, retrying..."
                                    sleep 10
                                    docker push ${DOCKER_IMAGE}:latest
                                }
                                
                                docker logout
                            '''
                        }
                    }
                }
            }
        }
    }
    
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
            echo '❌ Pipeline a échoué!'
        }
    }
}
