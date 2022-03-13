properties([pipelineTriggers([githubPush()])])

pipeline {
    agent any
    triggers {
        githubPush()
    }
    tools { 
        maven 'Maven_3.8.4' 
        jdk 'openjdk_11' 
    }
    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                ''' 
            }
        }
        stage('SonarQube analysis') {
            steps {
                withSonarQubeEnv(credentialsId: 'sonarqube', installationName: 'SonarQube') {
                sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
                }
            }
        }
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('UploadtoNexus') {
            steps {
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                archiveArtifacts artifacts: 'target/java-tomcat-maven-example.war'
            }
        }
        stage('DeployToServer') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    //sh 'sudo systemctl stop tomcat && sudo chmod -R 777 /opt/tomcat/latest/webapps/java-tomcat-maven-example && rm -rf /opt/tomcat/latest/webapps/java-tomcat-maven-example && mv /var/lib/jenkins/workspace/simple-java-maven-app/target/java-tomcat-maven-example.war /opt/tomcat/latest/webapps && sudo systemctl start tomcat'
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'tomcat',
                                verbose: true,
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'target/java-tomcat-maven-example.war',
                                        removePrefix: 'target/',
                                        remoteDirectory: '/tmp',
                                        execCommand: 'sudo systemctl stop tomcat && sudo chmod -R 777 /opt/tomcat/latest/webapps/java-tomcat-maven-example && rm -rf /opt/tomcat/latest/webapps/java-tomcat-maven-example && mv /tmp/java-tomcat-maven-example.war /opt/tomcat/latest/webapps && sudo systemctl start tomcat'
                                    )
                                ]
                            )
                        ]
                    )
                }
            }
        }
    }
}