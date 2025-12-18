pipeline {
    agent any
    tools {
        maven 'Maven-3'
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
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests -B'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    sh '''
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=student-management \
                        -Dsonar.host.url=http://192.168.49.2:30900 \
                        -Dsonar.login=admin \
                        -Dsonar.password=admin
                    '''
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    retry(3) {
                        sh """
                            docker build -f Dockerfile.fast -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                            docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                        """
                    }
                }
            }
        }
        stage('Docker Push') {
            when {
                expression { currentBuild.currentResult == 'SUCCESS' }
            }
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        '''
                        sh """
                            docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                            docker push ${DOCKER_IMAGE}:latest
                        """
                        sh 'docker logout'
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                        echo "Deploying to Kubernetes..."
                        kubectl apply -f mysql-deployment.yaml -n devops
                        kubectl apply -f spring-deployment.yaml -n devops
                        
                        echo "Waiting for deployment to be ready..."
                        kubectl rollout status deployment/mysql -n devops --timeout=300s
                        kubectl rollout status deployment/spring-app -n devops --timeout=300s
                        
                        echo "Deployment successful!"
                        kubectl get pods -n devops
                        kubectl get svc -n devops
                    '''
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
            echo '❌ Pipeline échoué!'
        }
    }
}
