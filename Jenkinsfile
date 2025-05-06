pipeline {
    agent any

    environment {
        MVN_HOME = tool name: 'Maven_3.6.3', type: 'maven'
        TOMCAT_URL = 'http://tomcat:8080/manager/text/deploy?path=/demo-podman&war=file:/path/to/your_app.war&update=true' // 替換為正確的檔案路徑
        TOMCAT_USER = 'jenkins'
        TOMCAT_PASS = 'yourpassword' // 替換為正確的密碼
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/frenchfu/demo-podman.git', branch: 'main'
            }
        }
        stage('Build') {
            steps {
                sh "${MVN_HOME}/bin/mvn clean package -Pprod -Dmaven.test.skip=true"
            }
        }
        stage('Rename WAR') {
            steps {
                sh 'mv target/demo-0.0.1-SNAPSHOT.war target/demo-podman.war'
            }
        }
        stage('Deploy to Tomcat by HTTP') {
            steps {
                sh """
                curl -u ${TOMCAT_USER}:${TOMCAT_PASS} \\
                  --upload-file target/demo-podman.war \\
                  '${TOMCAT_URL}'
                """
            }
        }
    }
}
