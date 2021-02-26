properties([pipelineTriggers([githubPush()])])
import java.text.SimpleDateFormat

def BUCKET_NAME ="mp-docs-stg-unity-it-fileshare-test"
def AKAMAI_URL ="https://docs-multiplayer-stg.unity3d.com/"

pipeline {
   agent {
      label "iit-jenkins-slave"
   }

    stages {
      stage('Akamai CDN purge data') {
         steps {
            script{
               akamai_purge(AKAMAI_URL, "akamai-api-token")
            }
         }
      }
      stage('Install nodejs and yarn') {
         steps {
            sh 'curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -'
            sh 'echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list'
            sh 'curl -fsSL https://deb.nodesource.com/setup_14.x | bash -'
            sh 'apt-get update && apt-get install -y nodejs yarn'
         }
      }
      stage('prepare env and install docusaurus') {
         steps {
            sh 'rm -rf temp'
            sh 'npx @docusaurus/init@latest init temp classic'
            sh 'yarn install'
         }
      }
      stage('Fix case issue on linux') {
         steps {
            sh 'cp -a static/img/transport/Pipeline-stages-diagram.png static/img/transport/pipeline-stages-diagram.png'
         }
      }
      stage('Build documentation') {
         steps {
            sh 'yarn build'
         }
      }
      stage('Sync with bucket') {
         steps {
            script{
               sync_bucket(BUCKET_NAME, "sa-mp-docs")
            }
         }
      }
   }
}

def sync_bucket(BUCKET, CREDS) {
    /* gsutil rsync -d will delete everything in the bucket that is not in the build dir and will update everything else */
    withCredentials([file(credentialsId: CREDS, variable: 'SERVICEACCOUNT')]) {
      sh label: '', script: """
      gcloud auth activate-service-account --key-file ${SERVICEACCOUNT}
      gsutil ls gs://${BUCKET}/
      gsutil -m rsync -r -d build/ gs://${BUCKET}
      gsutil ls gs://${BUCKET}/
      """
     }
}

def akamai_purge(AKAMAI_URL, CREDS) {
    withCredentials([file(credentialsId: CREDS, variable: 'EDGERC')]) {
      sh label: '', script: """
      echo "${EDGERC}" >> ~/.edgerc
      ls -lah ~/.edgerc
      """
     }
}