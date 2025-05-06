pipeline {
    agent any

    environment {
        // 設定 Maven 路徑（如果有多個 Maven，請根據 Jenkins 安裝名稱調整）
        MVN_HOME = tool name: 'Maven_3.6.3', type: 'maven'
        // Tomcat 伺服器資訊
        TOMCAT_HOST = 'tomcat' // 這裡是主機名或IP，Jenkins要能連通
        TOMCAT_USER = 'tomcat_user' // Tomcat 伺服器用戶
        TOMCAT_PASS = 'tomcat_password' // 若用 scp 需設 SSH Key 或密碼
        TOMCAT_WEBAPPS = '/opt/tomcat/webapps' // Tomcat webapps 目錄
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/frenchfu/demo-podman.git'
            }
        }
        stage('Build') {
            steps {
                sh "${MVN_HOME}/bin/mvn clean package -Dmaven.test.skip=true"
            }
        }
        stage('Rename WAR') {
            steps {
                sh 'mv target/demo-0.0.1-SNAPSHOT.war target/demo-podman.war'
            }
        }
        stage('Deploy to Tomcat') {
            steps {
                // 這裡用 scp 複製 war 到 Tomcat，如果 Tomcat 跑在本機可直接 cp
                sh """
                scp target/demo-podman.war ${TOMCAT_USER}@${TOMCAT_HOST}:${TOMCAT_WEBAPPS}/
                """
                // 如果 Tomcat 跑在Jenkins同一台主機，可用：
                // sh 'cp target/demo-podman.war /opt/tomcat/webapps/'
            }
        }
    }
}
