node {
    def warFileName = "helloworld.war"
    def jbossDeployPath = "/home/ec2-user/jboss-eap-8.0/standalone/deployments/"
    def warFilePath = "${jbossDeployPath}${warFileName}"
    def jbossUrl = "http://jboss.sandbox163.opentlc.com:9990/management"

    try {
        stage('Ensure JBoss Deployment Folder Exists') {
            echo "Checking if JBoss deployment folder exists..."
            sh """
                if [ ! -d "${jbossDeployPath}" ]; then
                    echo "Deployment folder not found! Creating..."
                    sudo mkdir -p ${jbossDeployPath}
                    sudo chown -R jenkins:jenkins ${jbossDeployPath}
                    sudo chmod -R 775 ${jbossDeployPath}
                else
                    echo "JBoss deployment folder exists."
                fi
            """
        }

        stage('Copy WAR to JBoss Deployment Folder') {
            echo "Copying WAR file to JBoss deployments directory..."
            sh "cp '/home/ubuntu/${warFileName}' '${warFilePath}'"
            sh "ls -lah ${jbossDeployPath}"
        }

        stage('Trigger Deployment in JBoss') {
            echo "Triggering deployment in JBoss..."
            sh "touch '${warFilePath}.dodeploy'"
        }

    } catch (Exception e) {
        currentBuild.result = 'FAILURE'
        throw e
    }
}
