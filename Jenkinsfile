node {
    // Define environment variables
    def ARTIFACTORY_SERVER = 'https://artifactory.apps.cluster-6m5l5.sandbox1208.opentlc.com/artifactory'
    def ARTIFACTORY_REPO = 'jboss-file'
    def ARTIFACTORY_REPO_URL = 'https://artifactory.apps.cluster-6m5l5.sandbox1208.opentlc.com/artifactory/jboss-file'
    def WAR_FILE = '/var/lib/jenkins/workspace/test/target/calculator-1.0-SNAPSHOT.war' 
    def PROJECT = 'jboss'
    def JBOSS_URL = 'http://jboss.6m5l5.sandbox1208.opentlc.com:9990/management'

    try {
        // Stage: Checkout
        stage('Checkout') {
            git credentialsId: 'daffa-gitlab', 
                branch: 'main', 
                url: 'https://gitlab.apps.cluster-6m5l5.sandbox1208.opentlc.com/jboss/pipeline.git'
        }

        // Stage: Prepare
        stage('Prepare') {
            echo "Preparing Java Spring Boot"
            sh 'mvn clean install -DskipTests=true'

            if (!fileExists(WAR_FILE)) {
                error "WAR file not found at: $WAR_FILE"
            }

            echo "WAR file located at: $WAR_FILE"
        }

        // Stage: SonarQube Analysis
        stage('SonarQube Analysis') {
            echo "Running SonarQube Analysis"
            sh ''' 
                mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=jboss \
                    -Dsonar.host.url=https://sonarqube.apps.cluster-6m5l5.sandbox1208.opentlc.com \
                    -Dsonar.login=sqp_f848f32de9af6a1a0bd181bb5861e7a8355a07fb
            '''
        }

        // Stage: Push to JFrog Artifactory
        stage('Push to JFrog Artifactory') {
            echo "Pushing WAR file to JFrog Artifactory"
            
            def WAR_FILE_NAME = sh(script: "basename $WAR_FILE", returnStdout: true).trim()
            echo "Uploading WAR file: $WAR_FILE_NAME"
            
            withCredentials([usernamePassword(credentialsId: 'artifactory', usernameVariable: 'JFROG_USER', passwordVariable: 'JFROG_PASS')]) {
                sh ''' 
                    jf rt u "$WAR_FILE" "$ARTIFACTORY_REPO/$PROJECT/$BUILD_NUMBER/$WAR_FILE_NAME" --url $ARTIFACTORY_SERVER --user $JFROG_USER --password $JFROG_PASS
                '''
            }

            env.WAR_URL = "$ARTIFACTORY_SERVER/$ARTIFACTORY_REPO/$PROJECT/$BUILD_NUMBER/$WAR_FILE_NAME"
            echo "WAR file uploaded to: $WAR_URL"
        }

        // Stage: Deploy to JBoss
        stage('Deploy to JBoss') {
            echo "Deploying WAR file to JBoss"
            
            def WAR_FILE_NAME = sh(script: "basename $WAR_FILE", returnStdout: true).trim()

            withCredentials([usernamePassword(credentialsId: 'jboss', usernameVariable: 'JBOSS_USER', passwordVariable: 'JBOSS_PASS')]) {
                sh '''
                    curl -u $JBOSS_USER:$JBOSS_PASS \
                        -X POST $JBOSS_URL/deployments \
                        -d '{"address":["deployment", "$WAR_FILE_NAME"],"operation":"add","content":[{"archive":"'$WAR_FILE'"}]}'
                '''
            }
            
            echo "WAR file deployed to JBoss at $JBOSS_URL"
        }

    } catch (Exception e) {
        // Handle failure
        currentBuild.result = 'FAILURE'
        throw e
    } finally {
        // Optionally, perform post-processing steps here
    }
}
