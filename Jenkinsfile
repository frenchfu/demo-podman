pipeline {
    agent any

    environment {
        MVN_HOME = tool name: 'Maven_3.6.3', type: 'maven'
        WILDFLY_HOST = 'wildfly'
        WILDFLY_MANAGEMENT_PORT = '9990'
        WILDFLY_USER = 'jenkins'
        WILDFLY_PASS = '1qaz@WSX3edc'
        APP_NAME = 'demo-podman.war'
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
                    // 1. 上傳 WAR 檔案，取得 BYTES_VALUE
                    def uploadResponse = sh(
                        script: """
                        curl --digest -s -u ${WILDFLY_USER}:${WILDFLY_PASS} \\
                             --form file=@target/${APP_NAME} \\
                             http://${WILDFLY_HOST}:${WILDFLY_MANAGEMENT_PORT}/management/add-content
                        """,
                        returnStdout: true
                    ).trim()

                    echo "Upload response: ${uploadResponse}"

                    def json = readJSON text: uploadResponse
                    def hash = json.result['BYTES_VALUE']

                    echo "Content hash: ${hash}"

                    // 2. 嘗試新增部署（add operation）
                    def addDeployPayload = """
                    {
                      "address": [{"deployment": "${APP_NAME}"}],
                      "operation": "add",
                      "content": [{"hash": {"BYTES_VALUE": "${hash}"}}],
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
