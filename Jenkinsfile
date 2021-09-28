pipeline {
    agent any

    options {
        timestamps() // Add timestamps to logging
        timeout(time: 10, unit: 'MINUTES') // Abort pipleine

        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
        disableConcurrentBuilds()
    }
    environment {
        PATH = "/usr/local/bin:$PATH"
    }
    parameters {
        choice(name: 'DeleteAnalysisData',
               choices: [
                   '0%',
                   '25%',
                   '50%',
                   '75%',
                   '100%'
               ],
               description: 'Select the amount of OLDEST Analysis data to delete [Note: TM Reports only get deleted using 100% setting]')
    }

    stages {

        stage('Init') {
            steps {
                echo 'Stage: Init'
                echo "branch=${env.BRANCH_NAME}, DeleteAnalysisData=${params.DeleteAnalysisData}"
            }
        }
        stage('Common Config') {
            steps {
                echo 'Stage: Common Config'
                // Checkout repo with common config files/scripts to 'common' folder
                checkout([$class: 'GitSCM',
                          branches: [[name: '*/main']],
                          doGenerateSubmoduleConfigurations: false,
                          extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'common']],
                          submoduleCfg: [], userRemoteConfigs: [[url: 'file:///home/jenkins/gitrepos/cicd-common']]
                         ])
            }
        }
        stage('Delete Analysis Results') {
            steps {
                echo 'Stage: Delete Analysis Results'
                dir('common') {
                    script {
                        // Chop of % sign (25% -> 25)
                        def pct = "${params.DeleteAnalysisData}"
                        sh """
                        ./am_run_static_results.sh ${pct.substring(0, pct.length()-1)}
                        """
                    }
                }
            }
        }
        
    }
    post {
        always {
            echo "post/always"
            deleteDir() // clean workspace
        }
        success {
            echo "post/success"
        }
        failure {
            echo "post/failure"
        }
    }
}
