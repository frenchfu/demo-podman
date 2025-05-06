pipeline {
    agent any

    environment {
        MVN_HOME = tool name: 'Maven_3.6.3', type: 'maven'
        TOMCAT_URL = 'http://tomcat:8080/manager/text/deploy?path=/demo-podman&update=true'
        TOMCAT_USER = 'jenkins'
        TOMCAT_PASS = 'yourpassword'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/frenchfu/demo-podman.git'
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
