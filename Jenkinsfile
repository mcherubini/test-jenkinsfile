pipeline {
    agent any
    environment{
        BITBUCKET_MICHELE_CREDENTIALS = credentials('michele-bitbucket-credentials')
    }
    stages{
        stage("clean repositority and download from git scm") {
            steps {
                sh "rm -rf api-java-spring"
                sh "git clone https://$BITBUCKET_MICHELE_CREDENTIALS_USR:$BITBUCKET_MICHELE_CREDENTIALS_PSW@bitbucket.org/racoon_royale/api-java-spring.git"
            }
        }
        stage("build") {
            steps {
                sh "mvn clean package"
            }
        }
        stage("run test and sonar"){
                environment {
                    scannerHome = tool 'SonarQube Scanner 4.0'
                }
                steps{
                    withSonarQubeEnv('sonarqube') {
                        sh "${scannerHome}/bin/sonar-scanner"
                        sh "mvn test"
                    }
                }
                /*steps("run test") {
                    sh "mvn test"
                }*/
        }
        stage("Quality Gate"){
            timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
                def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
                if (qg.status != 'OK') {
                    error "Pipeline aborted due to quality gate failure: ${qg.status}"
                }
            }
        }
        stage("Run test"){
            steps{
                sh "mvn test"
            }
        }
    }
}