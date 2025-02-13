node {
    def WAR_FILE = '/var/lib/jenkins/workspace/u-buntu/helloworld.war'
    def JBOSS_URL = 'http://jboss.sandbox163.opentlc.com:9990/management'
    def WAR_FILE_NAME = 'helloworld.war'

    try {
        // Checkout SCM
        stage('Checkout') {
            echo 'Checking out the repository...'
            checkout scm
        }

        // Copy WAR to Workspace
        stage('Copy WAR to Workspace') {
            echo 'Copying WAR file to workspace...'
            if (fileExists('/home/ubuntu/helloworld.war')) {
                echo 'WAR file exists at /home/ubuntu/helloworld.war. Copying to workspace.'
                sh "cp /home/ubuntu/helloworld.war ${WAR_FILE}"
                echo "WAR file successfully copied to workspace: ${WAR_FILE}"
            } else {
                error "WAR file does not exist at /home/ubuntu/helloworld.war"
            }
        }

        // Deploy WAR to JBoss
        stage('Deploy WAR to JBoss') {
            echo 'Deploying WAR file to JBoss...'
            withCredentials([usernamePassword(credentialsId: 'jboss-password', passwordVariable: 'JBOSS_PASSWORD', usernameVariable: 'JBOSS_USERNAME')]) {
                def curlCommand = """
                    curl --digest -u ${JBOSS_USERNAME}:${JBOSS_PASSWORD} \\
                         -X POST \\
                         -H "Content-Type: application/json" \\
                         -d '{
                               "operation": "composite",
                               "address": [],
                               "steps": [
                                   {
                                       "operation": "add",
                                       "address": [{"deployment": "${WAR_FILE_NAME}"}],
                                       "content": [
                                           {"archive": "${WAR_FILE}"}
                                       ],
                                       "enabled": true
                                   },
                                   {
                                       "operation": "deploy",
                                       "address": [{"deployment": "${WAR_FILE_NAME}"}]
                                   }
                               ]
                             }' \\
                         ${JBOSS_URL}
                """
                echo "Executing curl command: ${curlCommand}"
                sh curlCommand
            }
        }

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
