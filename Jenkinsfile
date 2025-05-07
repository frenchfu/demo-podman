pipeline {
    agent any

    environment {
        MVN_HOME = tool name: 'Maven_3.6.3', type: 'maven'
        WILDFLY_HOST = 'wildfly'               // Podman 網路中 WildFly 容器名稱或域名
        WILDFLY_MANAGEMENT_PORT = '9990'       // WildFly 管理介面預設埠
        WILDFLY_USER = 'jenkins'                  // WildFly 管理用戶名，請替換成你的
        WILDFLY_PASS = '1qaz@WSX3edc'           // WildFly 管理密碼，請替換成你的
        APP_NAME = 'demo-podman.war'            // 部署的 WAR 名稱
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/frenchfu/demo-podman.git', branch: 'main_for_jboss'
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
                    // 1. 上傳 WAR 檔案，取得 content hash
                    def uploadResponse = sh(
                        script: """
                        curl --digest -s -u ${WILDFLY_USER}:${WILDFLY_PASS} \\
                             -H "Content-Type: application/octet-stream" \\
                             --data-binary @target/${APP_NAME} \\
                             http://${WILDFLY_HOST}:${WILDFLY_MANAGEMENT_PORT}/management/add-content
                        """,
                        returnStdout: true
                    ).trim()

                    echo "Upload response: ${uploadResponse}"

                    def json = readJSON text: uploadResponse
                    def hash = json.result['BYTES_VALUE']

                    // hash 是陣列格式，需轉成字串陣列格式 JSON 才能用
                    def hashJson = groovy.json.JsonOutput.toJson(hash)

                    echo "Content hash: ${hashJson}"

                    // 2. 嘗試新增部署（add operation）
                    def addDeployPayload = """
                    {
                      "address": [{"deployment": "${APP_NAME}"}],
                      "operation": "add",
                      "content": [{"hash": ${hashJson}}],
                      "enabled": true,
                      "runtime-name": "${APP_NAME}"
                    }
                    """

                    def addResponse = sh(
                        script: """
                        curl --digest -s -u ${WILDFLY_USER}:${WILDFLY_PASS} -H "Content-Type: application/json" -d '${addDeployPayload}' \\
                        http://${WILDFLY_HOST}:${WILDFLY_MANAGEMENT_PORT}/management
                        """,
                        returnStdout: true
                    ).trim()

                    echo "Add deploy response: ${addResponse}"

                    def addJson = readJSON text: addResponse

                    // 3. 如果 add 失敗（例如部署已存在），改用 redeploy
                    if (addJson.outcome != 'success') {
                        echo "Add deployment failed, try redeploy..."

                        def redeployPayload = """
                        {
                          "address": [{"deployment": "${APP_NAME}"}],
                          "operation": "redeploy"
                        }
                        """

                        def redeployResponse = sh(
                            script: """
                            curl --digest -s -u ${WILDFLY_USER}:${WILDFLY_PASS} -H "Content-Type: application/json" -d '${redeployPayload}' \\
                            http://${WILDFLY_HOST}:${WILDFLY_MANAGEMENT_PORT}/management
                            """,
                            returnStdout: true
                        ).trim()

                        echo "Redeploy response: ${redeployResponse}"
                    } else {
                        echo "Deployment added successfully."
                    }
                }
            }
        }
    }
}
