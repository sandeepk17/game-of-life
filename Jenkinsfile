pipeline {
  agent {
      // Set Build Agent as Docker file 
      dockerfile true
  }
  environment {
      // Set env variables for Pipeline
      IMAGE = readMavenPom().getArtifactId()
      VERSION = readMavenPom().getVersion()
      ARTIFACTORY_SERVER_ID = "Artifactory1"
      ARTIFACTORY_URL = "http://192.168.0.104:8082/artifactory"
      ARTIFACTORY_CREDENTIALS = "admin.jfrog"
      CURRENT_BUILD_NO = "${currentBuild.number}"
      RELEASE_TAG = "${currentBuild.number}-${VERSION}"
      CURRENT_BRANCH = "${env.BRANCH_NAME}"
      OCTOHOME = "${OCTO_HOME}"
      properties = ""
      octopusURL = "https://rasmimr.octopus.app/"
  }

  stages {
    stage ('Environmental Variables') {
      steps {
          sh '''
              echo "PATH = ${PATH}"
              echo "${OCTOHOME}"
              echo "M2_HOME = ${M2_HOME}"
              echo "OCTO_HOME = ${OCTO_HOME}"
          '''
          script {
              properties = readProperties file: 'Build.properties'
              echo "${properties.testversion}"
              //env['testversion'] = props['testversion'];
              //env['company'] = props['company1'];
          }
          sh 'echo "user = ${user}"'
          sh 'echo "${testversion}"'
          //sh "echo ${env.props['company2']}"
          //sh "echo ${props['version']}"
      }
    }
    stage('Build and Deploy') {
      steps {
        echo "${VERSION}"
        echo "${IMAGE}"
        echo "${properties.user}"
        echo "${properties.company1}"
        echo "${properties.company2}"
        echo "${properties.company3}"
        echo "${properties.testversion}"
        echo "${CURRENT_BRANCH}"
        echo "$WORKSPACE"
        sh 'mvn clean deploy -s settings.xml'
      }
    }
    stage ('Archive Files') {
      steps {
          sh 'rm -rf dist'
          sh 'mkdir dist'
          fileOperations([
              fileCopyOperation(
                  flattenFiles: true, 
                  includes: "gameoflife-web/target/*.war", 
                  targetLocation: "$WORKSPACE/dist"), 
              fileCopyOperation(
                  flattenFiles: true, 
                  includes: "$WORKSPACE/scripts/*.sh", 
                  targetLocation: "$WORKSPACE/dist") 
          ])
      }
    }
    stage ('Deploy to Octopus') {
      steps {
          echo " Deploy to artifactory"
          withCredentials([string(credentialsId: 'OctopusAPIkey', variable: 'APIKey')]) {
              sh 'octo help'
              sh 'octo pack --id="OctoWeb" --version="${RELEASE_TAG}" --basePath="$WORKSPACE/dist" --outFolder="$WORKSPACE"'
              sh 'octo push --package $WORKSPACE/OctoWeb."${RELEASE_TAG}".nupkg --replace-existing --server ${octopusURL} --apiKey ${APIKey}'
              //sh 'octo create-release --project "Java" --package="OctoWeb:${RELEASE_TAG}" --server ${octopusURL} --apiKey ${APIKey}'
              //sh 'octo deploy-release --project "Java" --version latest --deployto Dev --server ${octopusURL} --apiKey ${APIKey} --progress'
          }
          rtUpload (
              serverId: "${ARTIFACTORY_SERVER_ID}",
              spec: '''{
                  "files": [
                      {
                      "pattern": "$WORKSPACE/OctoWeb.${RELEASE_TAG}.nupkg",
                      "target": "octopus/OctoWeb.${RELEASE_TAG}.nupkg"
                      }
                  ]
              }''',
              buildName: "${env.JOB_NAME}",
              buildNumber: "${currentBuild.number}"
          )
          withCredentials([string(credentialsId: 'OctopusAPIkey', variable: 'APIKey')]) {
              //sh 'octo pack --id="OctoWeb" --version="${RELEASE_TAG}" --basePath="$WORKSPACE/dist" --outFolder="$WORKSPACE"'
              //sh 'octo push --package $WORKSPACE/OctoWeb."${RELEASE_TAG}".nupkg --replace-existing --server ${octopusURL} --apiKey ${apiKey}'
              sh 'octo create-release --project "Java" --package="OctoWeb:${RELEASE_TAG}" --server ${octopusURL} --apiKey ${APIKey}'
              sh 'octo deploy-release --project "Java" --version latest --deployto Dev --server ${octopusURL} --apiKey ${APIKey} --progress'
          }
      }
    }
  }
}
