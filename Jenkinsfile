pipeline {
  agent any
  environment {
    GITHUB_REPO_URL = "https://github.com/abhigiri07/E-commerce-project-springBoot.git"
    GIT_BRANCH = "master2"
    SSH_CRED_ID = "web-key"
    EC2_IP = "3.236.87.27"
    REMOTE_USER = "ec2-user"
    APP_PATH = "/opt/ecom-app"
    PROJECT_DIR = "JtProject"
  }
  stages {
    stage('Checkout Code') { steps { git url: "${GITHUB_REPO_URL}", branch: "${GIT_BRANCH}" } }
    stage('Build with Maven') {
      steps { dir("${PROJECT_DIR}") { sh "mvn -B clean package -DskipTests" } }
      post { success { archiveArtifacts artifacts: "${PROJECT_DIR}/target/*.jar", fingerprint: true } }
    }
    stage('Deploy to EC2') {
      steps {
        sshagent(credentials: [SSH_CRED_ID]) {
          script {
            def jarFile = sh(script: "ls ${PROJECT_DIR}/target/*.jar 2>/dev/null | head -n 1 || true", returnStdout: true).trim()
            if (!jarFile) { error "No JAR file found" }
            sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${EC2_IP} 'sudo mkdir -p ${APP_PATH}'"
            sh "scp -o StrictHostKeyChecking=no "${jarFile}" ${REMOTE_USER}@${EC2_IP}:"${APP_PATH}/app.jar""
            sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${EC2_IP} 'sudo systemctl restart ecom.service || true'"
          }
        }
      }
    }
  }
  post { always { deleteDir() } }
}
