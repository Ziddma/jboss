node {
    def warFileName = "helloworld.war"  // The name of the WAR file
    def jbossUrl = "http://jboss.sandbox163.opentlc.com:9990/management"  // JBoss management URL

    try {
        // Stage: Copy WAR to Workspace
        stage('Copy WAR to Workspace') {
            echo "Copying WAR file to workspace..."
            sh "cp '/home/ubuntu/helloworld.war' '${WORKSPACE}/${warFileName}'"
        }

        // Stage: Deploy WAR to JBoss
        stage('Deploy WAR to JBoss') {
            echo "Deploying WAR file to JBoss..."
            withCredentials([usernamePassword(credentialsId: 'jboss-credentials', usernameVariable: 'JBOSS_USERNAME', passwordVariable: 'JBOSS_PASSWORD')]) {
                sh """
                    curl --digest -u ${env.JBOSS_USERNAME}:${env.JBOSS_PASSWORD} \
                         -X POST \
                         -H "Content-Type: application/json" \
                         -d '{
                               "operation": "add",
                               "address": [{"deployment": "${warFileName}"}],
                               "content": [{"input-stream": "@${WORKSPACE}/${warFileName}"}],
                               "enabled": true
                             }' \
                         ${jbossUrl}
                """
            }
        }

        // Stage: Deploy WAR to JBoss (Activate)
        stage('Deploy WAR to JBoss (Activate)') {
            echo "Activating WAR deployment on JBoss..."
            withCredentials([usernamePassword(credentialsId: 'jboss-credentials', usernameVariable: 'JBOSS_USERNAME', passwordVariable: 'JBOSS_PASSWORD')]) {
                sh """
                    curl --digest -u ${env.JBOSS_USERNAME}:${env.JBOSS_PASSWORD} \
                         -X POST \
                         -H "Content-Type: application/json" \
                         -d '{
                               "operation": "deploy",
                               "address": [{"deployment": "${warFileName}"}]
                             }' \
                         ${jbossUrl}
                """
            }
        }

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
