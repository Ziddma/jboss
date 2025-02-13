node {
    // Environment variables
    def warFileName = "helloworld.war"
    def jbossUrl = "http://jboss.sandbox163.opentlc.com:9990/management"
    def warFilePath = "${WORKSPACE}/${warFileName}"

    try {
        // Stage: Copy WAR to Workspace
        stage('Copy WAR to Workspace') {
            echo "Copying WAR file to workspace..."
            
            // Check if the WAR file exists before copying
            if (fileExists("/home/ubuntu/helloworld.war")) {
                echo "WAR file exists at /home/ubuntu/helloworld.war. Copying to workspace."
                sh "cp /home/ubuntu/helloworld.war ${warFilePath}"
            } else {
                error "WAR file does not exist at /home/ubuntu/helloworld.war. Aborting pipeline."
            }

            // Verifying the WAR file exists in the workspace
            if (!fileExists(warFilePath)) {
                error "WAR file not found in workspace at: ${warFilePath}. Aborting pipeline."
            }
            echo "WAR file successfully copied to workspace: ${warFilePath}"
        }

        // Stage: Deploy WAR to JBoss
        stage('Deploy WAR to JBoss') {
            echo "Deploying WAR file to JBoss..."
            
            withCredentials([usernamePassword(credentialsId: 'jboss-credentials', usernameVariable: 'JBOSS_USERNAME', passwordVariable: 'JBOSS_PASSWORD')]) {
                try {
                    // Debugging curl command
                    echo "Starting JBoss deployment via curl"

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
                                           "address": [{"deployment": "${warFileName}"}],
                                           "content": [
                                               {"path": "${warFilePath}"}
                                           ],
                                           "enabled": true
                                       },
                                       {
                                           "operation": "deploy",
                                           "address": [{"deployment": "${warFileName}"}]
                                       }
                                   ]
                                 }' \\
                             ${jbossUrl}
                    """
                    
                    // Print the curl command for debugging purposes
                    echo "Executing curl command: ${curlCommand}"
                    
                    // Run curl command
                    def curlResponse = sh(script: curlCommand, returnStdout: true).trim()
                    
                    // Output the response for debugging
                    echo "Curl Response: ${curlResponse}"
                    
                    // Check for errors in the curl response
                    if (curlResponse.contains("error") || curlResponse.contains("failure")) {
                        error "Failed to deploy WAR to JBoss. Response: ${curlResponse}"
                    }
                    
                    echo "WAR file successfully deployed to JBoss."
                } catch (Exception e) {
                    echo "Error occurred during JBoss deployment: ${e.getMessage()}"
                    throw e  // Re-throw the exception to fail the build
                }
            }
        }
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e  // Ensure the pipeline fails properly
    } finally {
        echo "Pipeline finished"
    }
}
