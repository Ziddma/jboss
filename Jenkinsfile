node {
    def warFileName = "helloworld.war"  // The name of the WAR file
    def jbossUrl = "http://jboss.sandbox163.opentlc.com:9990/management"  // JBoss management URL

    try {
        // Stage: Copy WAR to Workspace
        stage('Copy WAR to Workspace') {
            echo "Copying WAR file to workspace..."
            sh "cp '/home/ubuntu/helloworld.war' '${WORKSPACE}/${warFileName}'"
        }

        // Stage: Deploy WAR to JBoss using content URL reference
stage('Deploy WAR to JBoss') {
    echo "Deploying WAR file to JBoss using content URL..."
    withCredentials([usernamePassword(credentialsId: 'jboss-credentials', usernameVariable: 'JBOSS_USERNAME', passwordVariable: 'JBOSS_PASSWORD')]) {
        sh """
            echo "Starting Deployment..."
            curl --digest -L -D - \
                 -X POST \
                 -H "Content-Type: application/json" \
                 -u ${env.JBOSS_USERNAME}:${env.JBOSS_PASSWORD} \
                 -d '{
                       "operation": "add",
                       "address": {"deployment": "${warFileName}"},
                       "content": [
                           {"url": "file:${WORKSPACE}/${warFileName}"}
                       ],
                       "enabled": true,
                       "json.pretty":1
                     }' \
                 ${jbossUrl}
            echo "Deployment Response:"
        """
    }
}


        // Stage: Deploy WAR to JBoss (Activate)
        stage('Deploy WAR to JBoss (Activate)') {
            echo "Activating WAR deployment on JBoss..."
            withCredentials([usernamePassword(credentialsId: 'jboss-credentials', usernameVariable: 'JBOSS_USERNAME', passwordVariable: 'JBOSS_PASSWORD')]) {
                sh """
                    echo "Activating Deployment..."
                    curl --digest -u ${env.JBOSS_USERNAME}:${env.JBOSS_PASSWORD} \
                         -X POST \
                         -H "Content-Type: application/json" \
                         -d '{
                               "operation": "deploy",
                               "address": [{"deployment": "${warFileName}"}]
                             }' \
                         ${jbossUrl}
                    echo "Activation Response:"
                """
            }
        }

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
