
//  Docker related values:
def dockerTeam     = "ci"
def dockerImage    = "ghe-backup"
def dockerRepo     = "pierone.stups.zalan.do"
def shortImageName = "$dockerRepo/$dockerTeam/$dockerImage"

//  Lizzy related values:
def awsRegion   = "eu-west-1"
def kioAppName  = "ghe-backup-automata"
//def senzaConfig = "cloud-kraken/production.yaml"

//  Deployment values generated by the pipeline:
def nextStackVersion = "cd${env.BUILD_NUMBER}"
def fullImageName
def imageVersion

properties([
    pipelineTriggers([
      [$class: "GitHubPushTrigger"]
    ])
  ])

stage("Test") {
  /*
  zalando specific jenkins job that tests the actual code,
  similar to https://github.com/zalando/ghe-backup/blob/master/.travis.yml
  */
  build job: 'ghe-backup/GithubEnterpriseBackupTestPipelineJob', propagate: true, wait: true
}


if (env.BRANCH_NAME == 'master') {

    node('kraken') {
        checkout scm
        def shortCommit = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
        fullImageName = "$shortImageName:$nextStackVersion-$shortCommit"
        imageVersion = "$nextStackVersion-$shortCommit"
        echo "New image name: $fullImageName"
        echo "New image version: imageVersion"
    }

    stage("Build and Push Docker") {
          /*
          zalando specific jenkins job that creates a docker image and pushes it to the zalando docker registry
          */
        build job: 'ghe-backup/GithubEnterpriseBackupBuildAndPushImagePipelineJob',
            parameters: [string(name: 'IMAGE_PATH', value: fullImageName)], propagate: true, wait: true
    }


    timeout(time: 60, unit: "MINUTES") {
        /*
        https://issues.jenkins-ci.org/browse/JENKINS-36543
        https://support.cloudbees.com/hc/en-us/articles/226554067-Pipeline-How-to-add-an-input-step-with-timeout-that-continues-if-timeout-is-reached-using-a-default-value
        */
        def userInput = input(
        id: 'Proceed1', message: 'Do you want to deploy?', parameters: [
            [$class: 'BooleanParameterDefinition', defaultValue: false, description: '', name: 'Please confirm you agree with deployment']
        ])

        if (userInput == true) {
            /*
            zalando specific jenkins job that releases the EBS volume to be prepared for the next CF stack
            */
            stage("release EBS volume for next stack") {
                build job: 'ghe-backup/GithubEnterpriseBackupDeleteExistingAutomataStack', propagate: true, wait: true
            }

            /*
            zalando specific jenkins job creates a new CF stack for the backup
            */
            stage("deployment") {
                build job: 'ghe-backup/GithubEnterpriseBackupDeployAutomataStack',
                    parameters: [string(name: 'IMAGE_PATH', value: fullImageName),
                                 string(name: 'SENZA_YAML_FILE', value: 'ghe-backup-ci.yaml'),
                                 string(name: 'KIO_URL', value: 'https://kio.stups.zalan.do'),
                                 string(name: 'KIO_APP', value: 'ghe-backup-automata'),
                                 string(name: 'TAG', value: nextStackVersion),
                                 string(name: 'IMAGE_VERSION', value: imageVersion)],
                     propagate: true, wait: true
            }
        }else{
            echo "Deployment canceled."
        }
    }
}
