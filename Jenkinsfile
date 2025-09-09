pipeline {
  agent any

  tools {
    maven 'M3'   // Jenkins Global Tool config -> Name: M3
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/Vamshi420/registration-app.git'
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
      steps {
        withSonarQubeEnv('sonarqube-server') {  // replace with your configured server name
          withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
            sh "mvn -B sonar:sonar -Dsonar.login=${SONAR_TOKEN}"
          }
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
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
      cleanWs()
    }
  }
}
