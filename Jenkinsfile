node {
    def warFileName = "helloworld.war"
    def warFilePath = "/var/lib/jenkins/workspace/u-buntu/${warFileName}"
    def jbossUrl = "http://jboss.sandbox163.opentlc.com:9990/management"

    try {
        stage('Copy WAR to Workspace') {
            echo "Copying WAR file to workspace..."
            sh "cp '/home/ubuntu/helloworld.war' '${warFilePath}'"
        }

        stage('Deploy WAR to JBoss using add-content API') {
            echo "Deploying WAR file to JBoss using add-content API..."
            withCredentials([usernamePassword(credentialsId: 'jboss-credentials', usernameVariable: 'JBOSS_USERNAME', passwordVariable: 'JBOSS_PASSWORD')]) {
                sh """
                    echo "Starting Deployment..."
                    curl --digest -u ${env.JBOSS_USERNAME}:${env.JBOSS_PASSWORD} \
                         -X POST \
                         -H "Content-Type: application/json" \
                         -d '{
                               "operation": "composite",
                               "address": [],
                               "steps": [
                                   {
                                       "operation": "add-content",
                                       "address": {"deployment": "${warFileName}"},
                                       "content": [{"input-stream-index": 0}]
                                   },
                                   {
                                       "operation": "deploy",
                                       "address": {"deployment": "${warFileName}"}
                                   }
                               ],
                               "json.pretty":1
                             }' \
                         --data-binary "@${warFilePath}" \
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
