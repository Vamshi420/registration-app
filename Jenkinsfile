pipeline {
    agent any

    environment {
        GIT_URL = 'https://github.com/Vamshi420/registration-app.git'
        SONARQUBE_NAME = 'SonarQube'  // 👈 Must match name under Manage Jenkins → Configure System → SonarQube servers
        SONAR_PROJECT_KEY = 'sonar'
        NEXUS_URL = 'http://3.110.84.200:8081'
        TOMCAT_URL = 'http://13.204.64.2:8080'
    }

    stages {

        stage('Checkout') {
            steps {
                echo "📦 Cloning source code from GitHub..."
                git url: "${GIT_URL}", branch: 'main'
            }
        }

        stage('Build with Maven') {
            steps {
                echo "⚙️ Building project using Maven..."
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "🔍 Running SonarQube analysis..."
                withSonarQubeEnv("${SONARQUBE_NAME}") {
                    sh '''
                        mvn sonar:sonar \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.host.url=${SONAR_HOST_URL}
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo "🚦 Checking SonarQube Quality Gate status..."
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Upload Artifact to Nexus') {
            steps {
                echo "📤 Uploading artifact to Nexus repository..."
                withCredentials([usernamePassword(credentialsId: 'nexus-creds', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh '''
                        echo "Creating temporary Maven settings.xml for Nexus credentials..."
                        cat > settings-temp.xml <<EOF
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                              https://maven.apache.org/xsd/settings-1.0.0.xsd">
  <servers>
    <server>
      <id>nexus-releases</id>
      <username>${NEXUS_USER}</username>
      <password>${NEXUS_PASS}</password>
    </server>
    <server>
      <id>nexus-snapshots</id>
      <username>${NEXUS_USER}</username>
      <password>${NEXUS_PASS}</password>
    </server>
  </servers>
</settings>
EOF

                        mvn -B -s settings-temp.xml deploy -DskipTests
                        rm -f settings-temp.xml
                    '''
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo "🚀 Deploying WAR file to Tomcat..."
                withCredentials([usernamePassword(credentialsId: 'tomcat-creds', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PASS')]) {
                    sh '''
                        WAR_FILE=$(find . -name "*.war" | head -n 1)
                        echo "Deploying $WAR_FILE to Tomcat at ${TOMCAT_URL}"
                        curl -u ${TOMCAT_USER}:${TOMCAT_PASS} --upload-file $WAR_FILE "${TOMCAT_URL}/manager/text/deploy?path=/registration-app&update=true"
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "📜 Pipeline execution finished!"
        }
        success {
            echo "✅ CI/CD Pipeline SUCCESS — Build, Analysis, Upload, Deploy done!"
        }
        failure {
            echo "❌ Pipeline FAILED — Check above logs for details."
        }
    }
}
