def curlDeploy = """
    curl --digest -u ${JBOSS_USERNAME}:${JBOSS_PASSWORD} \\
         -X POST \\
         -H "Content-Type: application/json" \\
         -d '{
               "operation": "composite",
               "address": [],
               "steps": [
                   {
                       "operation": "add",
                       "address": [{"deployment": "${warFileName}"}],
                       "content": [
                           {
                               "archive": "${warFilePath}",
                               "path": "/path/on/jboss/server/${warFileName}"
                           }
                       ],
                       "enabled": true
                   },
                   {
                       "operation": "deploy",
                       "address": [{"deployment": "${warFileName}"}]
                   }
               ]
             }' \\
         ${jbossUrl}
"""
