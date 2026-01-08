pipeline {
    agent any

    environment {
        scannerHome = tool 'sonar8.0'
    }

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Archiving WAR artifact...'
                    archiveArtifacts artifacts: 'target/*.war'
                }
            }
        }

        stage('UNIT TEST') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('sonarqubeserver') {
                    sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=java-tomcat-sample \
                        -Dsonar.projectName=java-tomcat-sample \
                        -Dsonar.projectVersion=4.0 \
                        -Dsonar.sources=src \
                        -Dsonar.junit.reportsPath=target/surefire-reports \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                    """
                }
            }
        }

        stage('UploadArtifact') {
            steps {
                script {
                    // Maven-safe version (NO spaces, NO colons)
                    def timestamp = new Date().format("yyyyMMdd-HHmmss", TimeZone.getTimeZone('UTC'))
                    def artifactVersion = "2-${timestamp}"

                    echo "Uploading artifact version: ${artifactVersion}"

                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: '172.31.81.203:8081',
                        groupId: 'QA',
                        version: artifactVersion,
                        repository: 'myjavaapp',
                        credentialsId: 'sonartypecred',
                        artifacts: [
                            [
                                artifactId: 'java-tomcat-sample',
                                classifier: '',
                                file: 'target/java-tomcat-maven-example.war',
                                type: 'war'
                            ]
                        ]
                    )
                }
            }
        }
    }
}
