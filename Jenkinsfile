pipeline {
    agent any
    environment {
        WAR_FILE = '/var/lib/jenkins/workspace/u-buntu/helloworld.war'
        JBOSS_URL = 'http://jboss.sandbox163.opentlc.com:9990/management'
        WAR_FILE_NAME = 'helloworld.war'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Copy WAR to Workspace') {
            steps {
                echo 'Copying WAR file to workspace...'
                script {
                    if (fileExists('/home/ubuntu/helloworld.war')) {
                        echo 'WAR file exists at /home/ubuntu/helloworld.war. Copying to workspace.'
                        sh "cp /home/ubuntu/helloworld.war ${env.WAR_FILE}"
                        echo "WAR file successfully copied to workspace: ${env.WAR_FILE}"
                    } else {
                        error "WAR file does not exist at /home/ubuntu/helloworld.war"
                    }
                }
            }
        }

        stage('Deploy WAR to JBoss') {
            steps {
                echo 'Deploying WAR file to JBoss...'
                withCredentials([usernamePassword(credentialsId: 'jboss-password', passwordVariable: 'JBOSS_PASSWORD', usernameVariable: 'JBOSS_USERNAME')]) {
                    script {
                        def curlCommand = """
                            curl --digest -u ${env.JBOSS_USERNAME}:${env.JBOSS_PASSWORD} \\
                                 -X POST \\
                                 -H "Content-Type: application/json" \\
                                 -d '{
                                       "operation": "composite",
                                       "address": [],
                                       "steps": [
                                           {
                                               "operation": "add",
                                               "address": [{"deployment": "${env.WAR_FILE_NAME}"}],
                                               "content": [
                                                   {"archive": "${env.WAR_FILE}"}
                                               ],
                                               "enabled": true
                                           },
                                           {
                                               "operation": "deploy",
                                               "address": [{"deployment": "${env.WAR_FILE_NAME}"}]
                                           }
                                       ]
                                     }' \\
                                 ${env.JBOSS_URL}
                        """
                        echo "Executing curl command: ${curlCommand}"
                        sh curlCommand
                    }
                }
            }
        }
    }
}
