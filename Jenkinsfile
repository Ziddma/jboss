pipeline {
    agent any

    environment {
        warFileName = "helloworld.war"
        jbossUrl = "http://jboss.sandbox163.opentlc.com:9990/management"
    }

    stages {
        stage('Copy WAR to Workspace') {
            steps {
                script {
                    // Copy the WAR file to Jenkins workspace
                    sh 'cp "/home/ubuntu/helloworld.war" "${WORKSPACE}/${warFileName}"'
                }
            }
        }

        stage('Deploy WAR to JBoss') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jboss-credentials', usernameVariable: 'JBOSS_USERNAME', passwordVariable: 'JBOSS_PASSWORD')]) {
                    script {
                        // Deploy WAR to JBoss using Digest authentication with PUT method
                        sh """
                            curl --digest -u ${JBOSS_USERNAME}:${JBOSS_PASSWORD} \
                                 -X PUT \
                                 -H "Content-Type: application/json" \
                                 -d '{
                                       "operation": "composite",
                                       "address": [],
                                       "steps": [
                                           {
                                               "operation": "add",
                                               "address": [{"deployment": "${warFileName}"}],
                                               "content": [
                                                   {"path": "${warFileName}"}
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
            }
        }
    }
}