pipeline {
    agent any

    tools {
        maven 'MAVEN_HOME'   // Configure this in Jenkins Global Tool Configuration
        jdk 'JAVA_HOME'      // Configure this in Jenkins Global Tool Configuration
    }

    environment {
        // 👇 Replace with the exact SonarQube server name you set in Manage Jenkins → Configure System → SonarQube servers
        SONARQUBE_ENV = 'sonarqube-server'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Vamshi420/registration-app.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'webapp/target/*.war', fingerprint: true
                }
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'tomcat-credentials', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    sh '''
                        curl -u $TOMCAT_USER:$TOMCAT_PASS \
                        --upload-file webapp/target/webapp.war \
                        "http://13.235.254.177:8080/manager/text/deploy?path=/registration-app&update=true"
                    '''
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        failure {
            echo 'Pipeline failed.'
        }
        success {
            echo 'Pipeline succeeded.'
        }
    }
}
