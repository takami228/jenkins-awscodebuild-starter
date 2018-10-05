pipeline {
    agent any

    post {
        success {
            mattermostSend color: "good", message: ":smile: [${env.JOB_NAME}](${env.BUILD_URL}) was succeeded."
        }
        failure {
            updateGitlabCommitStatus name: 'codebuild', state: 'failed'
            mattermostSend color: "danger", message: ":dizzy_face: [${env.JOB_NAME}](${env.BUILD_URL}) was failed."
        }
    }

    options {
        gitLabConnection('GitLabConnectionName') // To Do
        gitlabBuilds(builds: ['codebuild'])
    }

    triggers {
        gitlab(
            triggerOnPush: true,
            triggerOnMergeRequest: true,
            branchFilterType: 'All',
            addNoteOnMergeRequest: true,
            addCiMessage: true
        )
    }

    stages {
        stage('codebuild') {
            steps {
                awsCodeBuild(
                    credentialsType: 'keys',
                    projectName: 'projectName', // To Do
                    region: 'aws-region', // To Do
                    sourceControlType: 'project',
                    sourceVersion: env.BRANCH_NAME,
                    envVariables: "[{BRANCH_NAME,${env.BRANCH_NAME}}]",
                )
            }
        }

        stage('gitlab') {
            steps {
                    updateGitlabCommitStatus name: 'codebuild', state: 'success'
                }
            }
        }

        stage('tag'){
            steps {
                script{
                    def releaseVersion = env.BRANCH_NAME.startsWith('release/') ? env.BRANCH_NAME.substring('release/'.length()) : null
                    if(releaseVersion){
                        mattermostSend color: "good", message: ":smile: ${releaseVersion} was released in S3"
                        def userRemoteConfig = scm.userRemoteConfigs.head()
                        withCredentials([usernameColonPassword(credentialsId: userRemoteConfig.credentialsId, variable: 'GIT_CREDENTIAL')]) {
                            def url = userRemoteConfig.url.replace('://', "://${env.GIT_CREDENTIAL}@")
                            sh 'git config user.email devops@example.com'
                            sh 'git config user.name devops'
                            sh "git tag -f $releaseVersion"
                            sh "git push $url $releaseVersion -f"
                            sh "git push $url --delete ${env.BRANCH_NAME}"
                        }
                    }
                }
            }
        }
    }
}
