import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles) {
    if (p['publishMethod'] == 'FTP') {
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
    }
  }
  return null  
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=737f2af1-397b-462d-8fc4-b03c2c14e71b',
           'AZURE_TENANT_ID=939b1169-0f64-450b-954a-01cfbf90b424']) {
    stage('init') {
      checkout scm
    }

    stage('build') {
      sh 'mvn clean package'
    }

    stage('deploy') {
      def resourceGroup = 'jenkins-get-started-rg'
      def webAppName = 'Rocky1228'
      
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
        sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
        '''
      }

      // get publish settings
      def pubProfilesJson = sh(script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true).trim()
      def ftpProfile = getFtpPublishProfile(pubProfilesJson)

      if (ftpProfile) {
        // upload package
        sh "az webapp deploy --resource-group ${resourceGroup} --name ${webAppName} --src-path target/calculator-1.0.war --type war"
      } else {
        error("FTP publish profile not found.")
      }

      // log out
      sh 'az logout'
    }
  }
}
