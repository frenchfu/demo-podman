pipeline {
    agent any

    environment {
        MVN_HOME = tool name: 'Maven_3.6.3', type: 'maven'
        WILDFLY_HOST = 'wildfly'               // Podman 網路中 WildFly 容器名稱或域名
        WILDFLY_MANAGEMENT_PORT = '9990'       // WildFly 管理介面預設埠
        WILDFLY_USER = 'admin'                  // WildFly 管理用戶名，請替換成你的
        WILDFLY_PASS = 'yourpassword'           // WildFly 管理密碼，請替換成你的
        APP_NAME = 'demo-podman.war'            // 部署的 WAR 名稱
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
        stage('Deploy to WildFly') {
            steps {
                script {
                    // 先上傳內容，取得 hash
                    def uploadResponse = sh(
                        script: """
                        curl -s -u ${WILDFLY_USER}:${WILDFLY_PASS} -H "Content-Type: application/octet-stream" \\
                             --data-binary @target/${APP_NAME} \\
                             http://${WILDFLY_HOST}:${WILDFLY_MANAGEMENT_PORT}/management/add-content
                        """,
                        returnStdout: true
                    ).trim()

                    echo "Upload response: ${uploadResponse}"

                    // 從回傳中擷取 hash
                    def json = readJSON text: uploadResponse
                    def hash = json.result['BYTES_VALUE']

                    // 部署或更新應用
                    sh """
                    curl -s -u ${WILDFLY_USER}:${WILDFLY_PASS} -H "Content-Type: application/json" -d '
                    {
                      "address": [{"deployment": "${APP_NAME}"}],
                      "operation": "add",
                      "content": [{"hash": ${hash}}],
                      "enabled": true,
                      "runtime-name": "${APP_NAME}"
                    }' http://${WILDFLY_HOST}:${WILDFLY_MANAGEMENT_PORT}/management || \\
                    curl -s -u ${WILDFLY_USER}:${WILDFLY_PASS} -H "Content-Type: application/json" -d '
                    {
                      "address": [{"deployment": "${APP_NAME}"}],
                      "operation": "redeploy"
                    }' http://${WILDFLY_HOST}:${WILDFLY_MANAGEMENT_PORT}/management
                    """
                }
            }
        }
    }
}
