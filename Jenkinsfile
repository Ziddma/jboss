node {
    def warFileName = "helloworld.war"  // Nama WAR file
    def warFilePath = "/home/ec2-user/jboss-eap-8.0/standalone/deployments/${warFileName}" // Path yang benar di dalam JBoss
    def jbossUrl = "http://jboss.sandbox163.opentlc.com:9990/management"  // URL JBoss API

    try {
        // **Stage 1: Copy WAR File to JBoss Deployments Folder**
        stage('Copy WAR to JBoss Deployment Folder') {
            echo "Copying WAR file to JBoss deployments directory..."
            sh "cp '/home/ubuntu/helloworld.war' '${warFilePath}'"

            // Verifikasi apakah file sudah berhasil disalin
            sh "ls -lah ${warFilePath}"
        }

        // **Stage 2: Check and Remove Previous Deployment (Jika Ada)**
        stage('Check & Remove Previous Deployment') {
            echo "Checking if deployment already exists..."
            withCredentials([usernamePassword(credentialsId: 'jboss-credentials', usernameVariable: 'JBOSS_USERNAME', passwordVariable: 'JBOSS_PASSWORD')]) {
                def checkDeployment = sh(script: """
                    curl --silent --digest -u ${env.JBOSS_USERNAME}:${env.JBOSS_PASSWORD} \
                         -X GET ${jbossUrl}/deployment=${warFileName}:read-resource?json.pretty=1
                """, returnStdout: true).trim()

                if (checkDeployment.contains("\"outcome\" : \"success\"")) {
                    echo "Deployment found! Removing previous deployment..."
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
                } else {
                    echo "Deployment does not exist. Proceeding with deployment."
                }
            }
        }

        // **Stage 3: Deploy WAR File to JBoss**
        stage('Deploy WAR to JBoss') {
            echo "Deploying WAR file to JBoss using deployment scanner..."
            sh "touch '${warFilePath}.dodeploy'"

            // Cek apakah marker file sudah ada untuk trigger deployment
            sh "ls -lah /home/ec2-user/jboss-eap-8.0/standalone/deployments/"
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
