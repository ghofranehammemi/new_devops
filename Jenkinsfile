pipeline {
    agent any
    
    tools {
        maven 'Maven-3'
        jdk 'JDK-17'
    }
    
    environment {
        SONAR_TOKEN = 'sqp_b765f177329e0e5f03e06ccbf925cf7907479886'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '========================================='
                echo 'Récupération du code'
                echo '========================================='
                git branch: 'main',
                    url: 'https://github.com/ghofranehammemi/new_devops.git'
            }
        }
        
        stage('Build & Test') {
            steps {
                echo '========================================='
                echo 'Build et Tests'
                echo '========================================='
                sh 'mvn clean verify'
            }
        }
        
        stage('SonarQube Analysis') {
    steps {
        script {
            sh '''
                mvn clean verify sonar:sonar \
                -Dsonar.projectKey=student-management \
                -Dsonar.host.url=http://localhost:9000 \
                -Dsonar.login=sqa_cdb6c27c66aa39bad6a66777ded332c88764ff4b
            '''
        }
    }
}
        
        stage('Package') {
            steps {
                echo '========================================='
                echo 'Packaging'
                echo '========================================='
                sh 'mvn package -DskipTests'
            }
        }
    }
    
    post {
        success {
            echo '========================================='
            echo '✅ BUILD SUCCESS !'
            echo 'Voir SonarQube: http://localhost:9000'
            echo '========================================='
        }
        failure {
            echo '========================================='
            echo '❌ BUILD FAILED !'
            echo '========================================='
        }
    }
}            
