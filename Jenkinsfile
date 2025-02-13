node {
    // Definisikan variabel
    def warFileName = "helloworld.war"
    def jbossUrl = "http://jboss.sandbox163.opentlc.com:9990/management"
    def warFilePath = "${WORKSPACE}/${warFileName}"

    try {
        stage('Copy WAR to Workspace') {
            echo "Copying WAR file to workspace"
            // Menyalin WAR ke workspace
            sh "cp /home/ubuntu/helloworld.war ${warFilePath}"
        }

        stage('Verify WAR File') {
            echo "WAR file path: ${warFilePath}"
            echo "Checking if WAR file exists in workspace"
            sh "ls -l ${WORKSPACE}"
            
            // Cek apakah file WAR benar-benar ada
            if (!fileExists(warFilePath)) {
                error "WAR file not found at: ${warFilePath}"
            }
        }

        stage('Deploy WAR to JBoss') {
            withCredentials([usernamePassword(credentialsId: 'jboss-credentials', usernameVariable: 'JBOSS_USERNAME', passwordVariable: 'JBOSS_PASSWORD')]) {
                echo "Deploying WAR file to JBoss"
                
                // Menjalankan curl untuk deploy
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
                                           {"archive": "${warFilePath}"}
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
                
                // Output curl command untuk debugging
                echo "Executing curl command: ${curlCommand}"

                // Eksekusi curl dan tangkap response untuk debugging
                def curlResponse = sh(script: curlCommand, returnStdout: true).trim()
                
                echo "Curl Response: ${curlResponse}"
                
                // Jika response curl mengandung error, lempar exception
                if (curlResponse.contains("error") || curlResponse.contains("failure")) {
                    error "Failed to deploy WAR to JBoss. Response: ${curlResponse}"
                }
            }
        }
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        echo "Pipeline finished"
    }
}
