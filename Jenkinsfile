pipeline {
    agent{
        kubernetes{
            defaultContainer 'jenkins-slave'
            yaml """
apiVersion: v1
kind: Pod
metadata:
   name: raccon-royale-pod
   labels: 
     app-racoon: racoon-v1
spec:
    volumes:
     - name: maven-m2-folder-volume
       hostPath:
       path: /host_mnt/c/Users/michele/.m2
     - name: sonarqube-conf
       hostPath:
        path:  //e/docker-containers/sonarqube/sonar.properties
        type: File
     - name: sonarqube-data
       hostPath:
        path: //e/docker-containers/sonarqube/database01/data
        type: Directory
     - name: mariadb-conf
       hostPath:
        path:  //e/docker-containers/mariadb/my.cnf
        type: File
    containers:
    - name: mariadb-database
      image: mariadb:latest
      env:
      - name: MYSQL_ROOT_PASSWORD
        value: "ViHasCome2"
      ports:
      - containerPort: 3306
      volumeMounts:
      - name: mariadb-conf
        mountPath: /etc/mysql/my.cnf
    - name: sonarqube-container
      image: sonarqube:latest
      ports:
      - containerPort: 9000
      volumeMounts:
      - name: sonarqube-conf
        mountPath: /opt/sonarqube/conf/sonar.properties
      - name: sonarqube-data
        mountPath: /opt/sonarqube/data
    - name: jenkins-slave
      image: jenkinsci/jnlp-slave
      workingDir: /home/jenkins
      volumeMounts:
      - name: maven-m2-folder-volume
        mountPath: /root/.m2
        command:
        - cat:
          tty: true
"""
        }
    }
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
                        withMaven(maven: 'Maven 3.6.1'){
                            //sh "mvn test"
                        }
                    }
                }
            }
            options{
                timeout(time: 1, unit: 'HOURS')  // Just in case something goes wrong, pipeline will be killed after a timeout
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