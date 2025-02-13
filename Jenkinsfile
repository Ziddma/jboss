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
                    sh 'cp "/home/ubuntu/helloworld.war" "${WORKSPACE}/${warFileName}"'
                }
            }
        }

        stage('Deploy WAR to JBoss') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jboss-credentials', usernameVariable: 'JBOSS_USERNAME', passwordVariable: 'JBOSS_PASSWORD')]) {
                    script {
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
                                                   {"path": "${WORKSPACE}/${warFileName}"}
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
