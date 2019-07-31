pipeline {
    agent { label 'master' }
    // Go to 'Manage Jenkins' -> 'Global Tool Configuration' and create the
    // following installations. If you give them different names you will need
    // to update the names in the 'tools' object below.
    // comment 6/28/19
    
    tools {
        maven 'maven-3.5.4'
        jdk 'jdk-8'
        nodejs "NodeJS"
    }
    environment {
        BUILD_COLOR = ""
        API_NAME = "apdev-examples-deploy"
        GET_BUILD_USER = sh(script: 'echo "${BUILD_USER_ID}"', returnStdout: true).trim()
		API_CLIENT_ID="123456789"
    }


    stages {
        stage('Pre-Build'){
            steps{
            
            sh 'printenv'
            sh 'ls'
            }
        }
        stage('Build'){
            steps{
                // M2_SETTINGS are special maven settings we have that include
                // the enterprise libraries required to build the application.
                // Add the maven file to your Jenkins server and create a env
                // variable in 'Manage Jenkins' -> 'Configure System' -> 'Global properties'
                // and create 'M2_SETTINGS' with a path to your settings.xml file.
                sh "echo ${M2_SETTINGS}"
                sh "echo ${BUILD_NUMBER}"
                sh "mvn release:update-versions -DdevelopmentVersion=1.0.${BUILD_NUMBER}-SNAPSHOT -s ${M2_SETTINGS}"
                sh "mvn -B clean verify -s ${M2_SETTINGS}"
            }

            post {
                success {
                    publishHTML target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'reports',
                        reportFiles: 'summary.html',
                        reportName: 'MUnit Report'
                    ]
                    publishHTML target: [
                        allowMissing: true,
                        alwaysLinkToLastBuild: false,
                        keepAll: true,
                        reportDir: 'target/site/jacoco',
                        reportFiles: 'index.html',
                        reportName: 'JUnit Report'
                    ]
                }
            }
        }

        stage('Deployment'){
            stages{
                stage('Cloudhub DEVELOPMENT - Deploy'){
                    environment {
                        ANYPOINT_USERNAME = "wiscarl" // This needs to change to a ci account
                        // https://support.cloudbees.com/hc/en-us/articles/203802500-Injecting-Secrets-into-Jenkins-Build-Jobs
                        ANYPOINT_PASSWORD = credentials('anypoint-password')
                    }
                    // when{
                    //     not {
                    //         changeRequest()
                    //     }
                    //     expression { GIT_BRANCH.matches(".*/master") && currentBuild.currentResult == 'SUCCESS' }
                    // }
                    steps{
                        // script{
                        //     ANYPOINT_ENV = ENV_MAPPING[RELEASE_ENVIRONMENT]['env']
                        // }
                        sh 'npm install anypoint-cli@latest'
                        // Original
                        //sh "./node_modules/anypoint-cli/src/app.js --environment=${ENV_MAPPING[RELEASE_ENVIRONMENT]['env']} runtime-mgr cloudhub-application modify ${ENV_MAPPING[RELEASE_ENVIRONMENT]['app']} target/${API_NAME}-1.0.${BUILD_NUMBER}-${RELEASE_ENVIRONMENT}-SNAPSHOT.zip"
                        // Test anypoint cli
                        //sh "./node_modules/anypoint-cli/src/app.js --environment='Development' runtime-mgr cloudhub-application list"
                        // Deploy the API
                        sh "./node_modules/anypoint-cli/src/app.js --environment='Development' runtime-mgr cloudhub-application modify ${API_NAME} target/${API_NAME}-1.0.${BUILD_NUMBER}-SNAPSHOT.zip "
                    }

                    post{
                      success {
                          script {
                              if (fileExists("reports")) {
                                  zip dir: "reports", zipFile: "munit-report.zip"
                              }
                              if (fileExists("target/site/jacoco")) {
                                  zip dir: "target/site/jacoco", zipFile: "junit-report.zip"
                              }
                              if (fileExists("reports") || fileExists("target/site/jacoco")) {
                                emailext (
                                // recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'CulpritsRecipientProvider']],
                                to: 'ckaufman@ra.rockwell.com',
                                //subject: "${JOB_BASE_NAME} | ${GIT_BRANCH}: Test Reports",
                                subject: 'munit/junit Reports',
                                body: "Test Reports attached",
                                attachmentsPattern: "*-report.zip"
                            )
                            }
                              stage "Create build output"
                              archiveArtifacts artifacts: 'target/**/*.zip', fingerprint: true
                          }
                      }
                    }
                }
            }
        }
      }
}
