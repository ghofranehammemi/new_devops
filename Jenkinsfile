pipeline {
    agent any
    
    tools {
        maven 'Maven'
      jdk 'JDK-17'

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
                sh 'mvn clean compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        
        // COMMENTÉ TEMPORAIREMENT - À configurer plus tard
        // stage('SonarQube Analysis') {
        //     steps {
        //         withSonarQubeEnv('SonarQube') {
        //             sh 'mvn sonar:sonar'
        //         }
        //     }
        // }
        
        stage('Package') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    sh """
                        docker build -t ghofranehammemi/student-management:${BUILD_NUMBER} .
                        docker tag ghofranehammemi/student-management:${BUILD_NUMBER} ghofranehammemi/student-management:latest
                    """
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
                        sh '''
                            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                            docker push ghofranehammemi/student-management:${BUILD_NUMBER}
                            docker push ghofranehammemi/student-management:latest
                            docker logout
                        '''
                    }
                }
            }
        }
    }
    
    post {
        always {
            sh """
                docker rmi ghofranehammemi/student-management:${BUILD_NUMBER} || true
                docker rmi ghofranehammemi/student-management:latest || true
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
