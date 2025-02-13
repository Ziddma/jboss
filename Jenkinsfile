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
                // Add the deployment to JBoss
                def addDeploymentResponse = sh(script: """
                    curl --digest -u ${env.JBOSS_USERNAME}:${env.JBOSS_PASSWORD} \
                         -X POST \
                         -H "Content-Type: application/json" \
                         -d '{
                               "operation": "add",
                               "address": [{"deployment": "${warFileName}"}],
                               "content": [{"archive": "@${WORKSPACE}/${warFileName}"}],
                               "enabled": true
                             }' \
                         ${jbossUrl}
                    """, returnStdout: true).trim()

                echo "Add Deployment Response: ${addDeploymentResponse}"

                // Check if the deployment was added successfully
                if (!addDeploymentResponse.contains('"outcome" : "success"')) {
                    error "Failed to add deployment. Response: ${addDeploymentResponse}"
                }
            }
        }

        // Stage: Deploy WAR to JBoss (Activate)
        stage('Deploy WAR to JBoss (Activate)') {
            echo "Activating WAR deployment on JBoss..."

            withCredentials([usernamePassword(credentialsId: 'jboss-credentials', usernameVariable: 'JBOSS_USERNAME', passwordVariable: 'JBOSS_PASSWORD')]) {
                // Activate the deployment
                def deployResponse = sh(script: """
                    curl --digest -u ${env.JBOSS_USERNAME}:${env.JBOSS_PASSWORD} \
                         -X POST \
                         -H "Content-Type: application/json" \
                         -d '{
                               "operation": "deploy",
                               "address": [{"deployment": "${warFileName}"}]
                             }' \
                         ${jbossUrl}
                    """, returnStdout: true).trim()

                echo "Deploy Response: ${deployResponse}"

                // Check if deployment was successful
                if (!deployResponse.contains('"outcome" : "success"')) {
                    error "Failed to deploy WAR file. Response: ${deployResponse}"
                }
            }
        }

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
