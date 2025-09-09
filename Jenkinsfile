pipeline {
  agent any

  environment {
    MAVEN_OPTS = '-Xmx1024m'
  }

  tools {
    maven 'M3'   // Ensure Maven is configured in Jenkins as 'M3'
  }

  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/Vamshi420/registration-app.git'
      }
    }

    stage('Build') {
      steps {
        sh 'mvn -B -DskipTests clean package'
      }
      post {
        always {
          archiveArtifacts artifacts: 'target/*.war', fingerprint: true
        }
      }
    }

    stage('Unit Tests') {
      steps {
        sh 'mvn test || true'
        junit testResults: 'target/surefire-reports/*.xml', allowEmptyResults: true
      }
    }

    stage('SonarQube Analysis') {
      environment {
        SONAR_HOST_URL = 'http://3.109.4.149:9000'
      }
      steps {
        withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
          sh "mvn -B sonar:sonar -Dsonar.host.url=${SONAR_HOST_URL} -Dsonar.login=${SONAR_TOKEN}"
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          waitForQualityGate(abortPipeline: true)
        }
      }
    }

    stage('Deploy to Tomcat') {
      steps {
        script {
          def war = sh(script: "ls target/*.war | head -n1", returnStdout: true).trim()
          withCredentials([usernamePassword(credentialsId: 'TOMCAT_CRED', usernameVariable: 'TOMCAT_USER', passwordVariable: 'TOMCAT_PSW')]) {
            sh "curl --fail --upload-file ${war} \"http://${TOMCAT_USER}:${TOMCAT_PSW}@13.235.254.177:8080/manager/text/deploy?path=/registration-app&update=true\""
          }
        }
      }
    }
  }

  post {
    success {
      echo 'Pipeline succeeded.'
    }
    failure {
      echo 'Pipeline failed.'
    }
    always {
      archiveArtifacts artifacts: 'target/*.war', allowEmptyArchive: true
    }
  }
}
