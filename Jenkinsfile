pipeline {
    agent any

    tools {
        maven 'maven'
        jdk 'jdk17'
    }

    environment {
        SONAR_TOKEN = credentials('sonarqube-token')    // Your SonarQube token ID
        TOMCAT_USER = "tomcat"
        TOMCAT_PASS = "tomcat"
        TOMCAT_URL  = "13.235.19.45:8080"
    }

    stages {

        stage('Git Checkout') {
            steps {
                git url: 'https://github.com/Vamshi420/registration-app.git', branch: 'main'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Server') {   // Must match EXACT Jenkins config name
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=registration-app \
                        -Dsonar.host.url=http://3.108.185.10:9000 \
                        -Dsonar.token=$SONAR_TOKEN
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo "Deploying WAR to Tomcat..."

                sh '''
                    WAR_FILE=$(ls webapp/target/*.war | head -n 1)

                    curl -u $TOMCAT_USER:$TOMCAT_PASS \
                        -T "$WAR_FILE" \
                        "http://$TOMCAT_URL/manager/text/deploy?path=/registration-app&update=true"
                '''
            }
        }
    }
}
