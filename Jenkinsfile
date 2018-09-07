pipeline {
    agent any

    post {
        success {
            updateGitlabCommitStatus name: 'build', state: 'success'
            updateGitlabCommitStatus name: 'test', state: 'success'
            mattermostSend color: "good", message: ":smile: [${env.JOB_NAME}](${env.BUILD_URL}) Build Success"
        }
        failure {
            updateGitlabCommitStatus name: 'build', state: 'failed'
            updateGitlabCommitStatus name: 'test', state: 'failed'
            mattermostSend color: "danger", message: ":dizzy_face: [${env.JOB_NAME}](${env.BUILD_URL}) Build Failed"
        }
    }

    options {
        gitLabConnection('GitLabConnectionName') // To Do
        gitlabBuilds(builds: ['test'])
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

        stage('tag') {
            steps {
                script{
                    def releaseVersion = env.BRANCH_NAME.startsWith('release/') ? env.BRANCH_NAME.substring('release/'.length()) : null
                    if(releaseVersion){
                        //Tag Push & Delete Branch
                        def userRemoteConfig = scm.userRemoteConfigs.head()
                        withCredentials([usernameColonPassword(credentialsId: userRemoteConfig.credentialsId, variable: 'GIT_CREDENTIAL')]) {
                            def url = userRemoteConfig.url.replace('://', "://${env.GIT_CREDENTIAL}@")
                            sh 'git config user.email gituser@example.com' // To Do
                            sh 'git config user.name gituser' // To Do
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
