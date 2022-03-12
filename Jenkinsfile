pipeline {
    agent any
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
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('UploadtoNexus') {
            steps {
                sh 'mvn clean deploy -Dmaven.test.skip=true'
            }
        }
        /*stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage('Deliver') {
            steps {
                sh './jenkins/scripts/deliver.sh'
            }
        }
        stage('SaveaAndUploadArtifact'){
            steps {
                archiveArtifacts artifacts: 'target/*.war'
                sh "curl -X 'POST' 'http://20.125.27.175:8081/service/rest/v1/components?repository=maven-snapshots' target/*" 
            }
        }*/
        stage('DeployToServer') {
            /*steps {     
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {    
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'tomcat',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                            transfers: [
                                sshTransfer(
                                    cleanRemote: false,
                                    excludes: '',
                                    execCommand: 'sudo systemctl stop tomcat && rm -rf /opt/tomcat/latest/webapps/sample && unzip /tmp/sample.war -d /opt/tomcat/latest/webapps && sudo systemctl start tomcat', 
                                    execTimeout: 120000,
                                    flatten: false,
                                    makeEmptyDirs: false,
                                    noDefaultExcludes: false,
                                    patternSeparator: '[, ]+',
                                    remoteDirectory: '/tmp',
                                    remoteDirectorySDF: false,
                                    removePrefix: 'target/',
                                    sourceFiles: 'target/sample.war')],
                                usePromotionTimestamp: false,
                                useWorkspaceInPromotion:false,
                                verbose: false
                            )
                        ]
                    )
                }
            }*/
            steps {
                withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
                    sshPublisher(
                        failOnError: true,
                        continueOnError: false,
                        publishers: [
                            sshPublisherDesc(
                                configName: 'tomcat',
                                sshCredentials: [
                                    username: "$USERNAME",
                                    encryptedPassphrase: "$USERPASS"
                                ], 
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: 'target/java-tomcat-maven-example.war',
                                        removePrefix: 'target/',
                                        remoteDirectory: '/tmp',
                                        execCommand: 'sudo systemctl stop tomcat && rm -rf /opt/tomcat/latest/webapps/java-tomcat-maven-example && unzip /tmp/java-tomcat-maven-example.war -d /opt/tomcat/latest/webapps && sudo systemctl start tomcat'
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