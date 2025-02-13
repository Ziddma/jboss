node {
    def warFileName = "helloworld.war"  // The name of the WAR file
    def jbossUrl = "http://jboss.sandbox163.opentlc.com:9990/management"  // JBoss management URL
    def warFilePath = "/home/ubuntu/${warFileName}"  // Path to the WAR file

    try {
        // Stage: Copy WAR to Workspace
        stage('Copy WAR to Workspace') {
            echo "Copying WAR file to workspace..."
            sh "cp '${warFilePath}' '${WORKSPACE}/${warFileName}'"
        }

        // Stage: Deploy WAR to JBoss
        stage('Deploy WAR to JBoss') {
            echo "Deploying WAR file to JBoss..."
            withCredentials([usernamePassword(credentialsId: 'jboss-credentials', usernameVariable: 'JBOSS_USERNAME', passwordVariable: 'JBOSS_PASSWORD')]) {
                sh """
                    echo "Starting Deployment..."
                    curl --digest -L -D - \
                         -u ${env.JBOSS_USERNAME}:${env.JBOSS_PASSWORD} \
                         -H "Content-Type: application/json" \
                         -d '{
                               "operation" : "composite",
                               "address" : [],
                               "steps" : [
                                   {"operation" : "add", "address" : {"deployment" : "${warFileName}"}, "content" : [{"url" : "file:${WORKSPACE}/${warFileName}"}]},
                                   {"operation" : "deploy", "address" : {"deployment" : "${warFileName}"}}
                               ],
                               "json.pretty": 1
                             }' \
                         ${jbossUrl}
                    echo "Deployment Response:"
                """
            }
        }

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
