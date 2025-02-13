node {
    // Definisikan variabel
    def warFileName = "helloworld.war"
    def jbossUrl = "http://jboss.sandbox163.opentlc.com:9990/management"

    try {
        stage('Copy WAR to Workspace') {
            echo "Copying WAR file to workspace"
            sh 'cp /home/ubuntu/helloworld.war "${WORKSPACE}/${warFileName}"'
        }

        stage('Deploy WAR to JBoss') {
            withCredentials([usernamePassword(credentialsId: 'jboss-credentials', usernameVariable: 'JBOSS_USERNAME', passwordVariable: 'JBOSS_PASSWORD')]) {
                echo "Deploying WAR file to JBoss"
                
                // Memastikan path yang benar untuk WAR
                def warFilePath = "${WORKSPACE}/${warFileName}"

                // Pastikan file ada sebelum melakukan deploy
                if (!fileExists(warFilePath)) {
                    error "WAR file not found at: ${warFilePath}"
                }

                // Jalankan curl untuk deploy
                sh """
                    curl --digest -u ${JBOSS_USERNAME}:${JBOSS_PASSWORD} \
                         -X POST \
                         -H "Content-Type: application/json" \
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
                             }' \
                         ${jbossUrl}
                """
            }
        }
    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        echo "Pipeline finished"
    }
}
