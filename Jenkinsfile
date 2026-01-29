pipeline {
    agent any

    tools {
        maven 'maven'
        jdk 'JDK17'
    }

    environment {
        TOMCAT_URL = 'http://3.111.36.198:8080'
        APP_NAME   = 'registration-app'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Vamshi420/registration-app.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh '''
                ls -l webapp/target
                curl -v -u tomcat:tomcat \
                -T webapp/target/*.war \
                "$TOMCAT_URL/manager/text/deploy?path=/$APP_NAME&update=true"
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
            echo "🌐 App URL: http://3.111.36.198:8080/${APP_NAME}"
        }
        failure {
            echo "❌ Build or Deployment failed"
        }
    }
}
