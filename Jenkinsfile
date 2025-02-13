pipeline {
    agent any

    environment {
        warFileName = "helloworld.war"
        jbossUrl = "http://jboss.sandbox163.opentlc.com:9990/management"
        jbossDeployDir = "/opt/jboss/wildfly/standalone/deployments" // JBoss deployment directory
    }

    stages {
        stage('Copy WAR to Workspace') {
            steps {
                script {
                    sh 'cp "/home/ubuntu/helloworld.war" "${WORKSPACE}/${warFileName}"'
                }
            }
        }

        stage('Deploy WAR to JBoss') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key-id', keyFileVariable: 'SSH_PRIVATE_KEY', usernameVariable: 'SSH_USER'),
                                 usernamePassword(credentialsId: 'jboss-credentials', usernameVariable: 'JBOSS_USERNAME', passwordVariable: 'JBOSS_PASSWORD')]) {
                    script {
                        // Ensure the SSH_PRIVATE_KEY is written to a file for use
                        writeFile file: '/tmp/ssh_key', text: SSH_PRIVATE_KEY
                        
                        // Set permissions for the private key file
                        sh 'chmod 600 /tmp/ssh_key'

                        // Add the SSH key to the agent
                        sh """
                            eval \$(ssh-agent -s)
                            ssh-add /tmp/ssh_key
                        """

                        // Deploy WAR using the JBoss Management API
                        sh """
                            curl --digest -u \${JBOSS_USERNAME}:\${JBOSS_PASSWORD} -X POST -H "Content-Type: application/json" -d '{
                                "operation": "composite",
                                "address": [],
                                "steps": [
                                    {
                                        "operation": "add",
                                        "address": [{"deployment": "${warFileName}"}],
                                        "content": [
                                            {"archive": "/var/lib/jenkins/workspace/${warFileName}"}
                                        ],
                                        "enabled": true
                                    },
                                    {
                                        "operation": "deploy",
                                        "address": [{"deployment": "${warFileName}"}]
                                    }
                                ]
                            }' ${jbossUrl}
                        """
                    }
                }
            }
        }
    }
}
