pipeline {
    agent { label 'jenkins-slave-npm' }

    environment {
        CI_CD_PROJECT = "${OPENSHIFT_BUILD_NAMESPACE}"
        //root of source code
        SOURCE_CONTEXT_DIR = ""

        //location of distribution files
        DIST_CONTEXT_DIR = "dist/"
        APP_NAME = "angular-unit-testing"

        // these are defaults that will help run openshift automation
        OCP_API_SERVER = "${OPENSHIFT_API_URL}"
        OCP_TOKEN = readFile('/var/run/secrets/kubernetes.io/serviceaccount/token').trim()
    }

    stages {

        stage('NPM Install and Test') {
            steps {
                slackSend ":start: $APP_NAME pipeline started. ${env.JOB_NAME} <${env.BUILD_URL}|${env.BUILD_NUMBER}>"
                sh "npm install"
                sh "npm run pipeline-test"
                sh "npm run pipeline-tslint"
            }
        }

        stage('Static Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonar-scanner-tool';
                    withSonarQubeEnv('sonar') {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }   

        stage('NPM Build') {
            steps {
                sh "npm run build"
            }
        }

        stage('Build Image') {
            steps {
                patchBuildConfigOutputLabels(env)
                script {
                    openshift.withCluster () {
                        openshift.startBuild( "${APP_NAME} --from-dir=${DIST_CONTEXT_DIR} -w" )
                    }
                }
            }
        }

        stage('Deploy: Dev'){
            agent { label 'jenkins-slave-ansible'}
            steps {
                script{
                    DEV_PROJECT = "${OPENSHIFT_BUILD_NAMESPACE}".reverse().drop(6).reverse()  + "-dev"
                    applyAnsibleInventory( 'dev' )
                    timeout(5) { // in minutes
                        openshift.loglevel(9)
                        promoteImageWithinCluster( "${CI_CD_PROJECT}/${APP_NAME}:latest", "${DEV_PROJECT}/${APP_NAME}", "deployed" )
                        verifyDeployment("${APP_NAME}", "${DEV_PROJECT}")
                    }
                }
            }
        }

        stage('Deploy: Demo'){
            agent { label 'jenkins-slave-ansible'}
            steps {
                script{
                    DEMO_PROJECT = "${OPENSHIFT_BUILD_NAMESPACE}".reverse().drop(6).reverse()  + "-demo"
                    applyAnsibleInventory( 'demo' )
                    timeout(5) { // in minutes
                        openshift.loglevel(9)
                        promoteImageWithinCluster( "${CI_CD_PROJECT}/${APP_NAME}:latest", "${DEMO_PROJECT}/${APP_NAME}", "deployed" )
                        verifyDeployment("${APP_NAME}", "${DEMO_PROJECT}")
                    }
                }
            }
        }
    }

    post {
        always {
            slackBuildResult(currentBuild.currentResult)
        }
    }
}

