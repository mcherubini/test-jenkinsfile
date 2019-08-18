pipeline {
    agent any
    environment{
        BITBUCKET_MICHELE_CREDENTIALS = credentials('michele-bitbucket-credentials')
        PROJECT_NAME = 'api-java-spring'
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
                withMaven(maven: 'Maven 3.6.1'){
                    dir("${env.WORKSPACE}/$PROJECT_NAME/"){
                        sh "mvn help:effective-settings"
                        sh "mvn -f pom.xml -Dmaven.test.skip=true clean package"
                    } 
                }
            }
        }
        stage("run test and sonar"){
            environment {
                scannerHome = tool 'SonarQube Scanner 4.0'
            }
            steps{
                withSonarQubeEnv('Sonarqube 7.x') {
                    dir("${env.WORKSPACE}/$PROJECT_NAME/"){
                        sh "${scannerHome}/bin/sonar-scanner -X -Dsonar.projectName='Demo project' -Dsonar.projectVersion=1.0" +
                        " -Dsonar.projectKey=com.example:demo -Dsonar.sources=src/main" +
                        " -Dsonar.tests=src/test -Dsonar.java.binaries=target/classes"
                    } 
                            //sh "mvn test"
                }
            }
            options{
                timeout(time: 1, unit: 'HOURS')  // Just in case something goes wrong, pipeline will be killed after a timeout
            }
                    steps("run test") {
                        sh "mvn test"
                    }
        }
        stage("Sonar Quality Gate"){
            steps{
                script{
                    def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
                    echo "Sonar Webhook received"
                    if (qg.status != 'OK') {
                        error "Pipeline aborted due to quality gate failure: ${qg.status}"
                    }
                }
            }
        }
    }
}