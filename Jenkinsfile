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
            parallel{
                environment {
                    scannerHome = tool 'SonarQube Scanner 4.0'
                }
                steps("Sonar scan"){
                    withSonarQubeEnv('sonarqube') {
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
                steps("run test") {
                    sh "mvn test"
                }
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}