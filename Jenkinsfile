node {
    def warFileName = "helloworld.war"  // Nama WAR file yang akan dideploy
    def warFilePath = "${WORKSPACE}/${warFileName}"  // Path ke file WAR di workspace Jenkins
    def jbossUrl = "http://jboss.sandbox163.opentlc.com:9990/management"  // JBoss Management API URL

    try {
        // **Stage 1: Copy WAR File to Jenkins Workspace**
        stage('Copy WAR to Workspace') {
            echo "Copying WAR file to workspace..."
            sh "cp '/home/ubuntu/helloworld.war' '${warFilePath}'"
            
            // Verifikasi apakah file WAR sudah berhasil disalin
            sh "ls -lah ${warFilePath}"
        }

        // **Stage 2: Undeploy Existing Deployment (Jika Ada)**
        stage('Undeploy Previous Deployment') {
            echo "Checking and removing existing deployment..."
            withCredentials([usernamePassword(credentialsId: 'jboss-credentials', usernameVariable: 'JBOSS_USERNAME', passwordVariable: 'JBOSS_PASSWORD')]) {
                sh """
                    curl --digest -L -D - \
                         -X POST \
                         -H "Content-Type: application/json" \
                         -u ${env.JBOSS_USERNAME}:${env.JBOSS_PASSWORD} \
                         -d '{
                               "operation" : "composite",
                               "address" : [],
                               "steps" : [
                                   {"operation" : "undeploy", "address" : {"deployment" : "${warFileName}"}},
                                   {"operation" : "remove", "address" : {"deployment" : "${warFileName}"}}
                               ],
                               "json.pretty":1
                             }' \
                         ${jbossUrl}
                """
            }
        }

        // **Stage 3: Deploy WAR File to JBoss Content Repository**
        stage('Deploy WAR to JBoss') {
            echo "Deploying WAR file to JBoss using content URL..."
            withCredentials([usernamePassword(credentialsId: 'jboss-credentials', usernameVariable: 'JBOSS_USERNAME', passwordVariable: 'JBOSS_PASSWORD')]) {
                sh """
                    curl --digest -L -D - \
                         -X POST \
                         -H "Content-Type: application/json" \
                         -u ${env.JBOSS_USERNAME}:${env.JBOSS_PASSWORD} \
                         -d '{
                               "operation" : "composite",
                               "address" : [],
                               "steps" : [
                                   {
                                       "operation" : "add",
                                       "address" : {"deployment" : "${warFileName}"},
                                       "content" : [{"url" : "file:${warFilePath}"}]
                                   },
                                   {
                                       "operation" : "deploy",
                                       "address" : {"deployment" : "${warFileName}"}
                                   }
                               ],
                               "json.pretty":1
                             }' \
                         ${jbossUrl}
                """
            }
        }

        // **Stage 4: Verify Deployment**
        stage('Verify Deployment') {
            echo "Verifying deployment status on JBoss..."
            withCredentials([usernamePassword(credentialsId: 'jboss-credentials', usernameVariable: 'JBOSS_USERNAME', passwordVariable: 'JBOSS_PASSWORD')]) {
                sh """
                    curl --digest -L -D - \
                         -X GET \
                         -H "Content-Type: application/json" \
                         -u ${env.JBOSS_USERNAME}:${env.JBOSS_PASSWORD} \
                         ${jbossUrl}/deployment=${warFileName}:read-resource?json.pretty=1
                """
            }
        }

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
