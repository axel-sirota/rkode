#!/usr/bin/env groovy

// Set up project specific configuration
def name                = 'anypoint-performance-framework'
def packagePath         = 'dist'

node('any') {

        stage('Checkout SCM') {
            checkout([
                $class: 'GitSCM',
                branches: [[name: "refs/heads/${env.BRANCH_NAME}"]],
                extensions: [[$class: 'LocalBranch']],
                userRemoteConfigs: scm.userRemoteConfigs
            ])
        }

        stage('Clean') {
            sh "rm -rf build anypoint_performance_framework.egg-info dist"
        }

        stage('Unit Tests') {
            timestamps {
                timeout(time: 30, unit: 'MINUTES') {
                    try {
                        sh 'nosetests tests/unit --with-xunit'
                    } finally {
                        step([$class: 'JUnitResultArchiver', testResults: 'nosetests-units.xml'])
                    }
                }
            }
        }

        stage('Code checking') {

            sh 'pylint --output-format=parseable --reports=no module > pylint.log || echo "pylint exited with $?")'
            sh 'cat render/pylint.log'

            step([
                $class                     : 'WarningsPublisher',
                parserConfigurations       : [[parserName: 'PYLint', pattern   : 'pylint.log']],
                unstableTotalAll           : '0',
                usePreviousBuildAsReference: true
            ])

//             sonarScan {
//                integrationBranch = 'develop'
//            }
        }

        stage('Build .whl & .tar.gz')
            sh 'python setup.py bdist_wheel'
            sh 'python setup.py sdist'

        stage 'Archive build artifact: .whl & .tar.gz'
            archive 'dist/*'

        stage 'Trigger downstream publish'
            build job: 'publish-local', parameters: [
                string(name: 'artifact_source', value: "${currentBuild.absoluteUrl}/artifact/dist/*zip*/dist.zip"),
                string(name: 'source_branch', value: "${env.BRANCH_NAME}")]
    }
}
