
pipeline {
    agent {
        label 'app'
    }
    environment {
        SERVER_ID = 'jfrog'
        MAVEN_OPTS = "--add-exports jdk.compiler/com.sun.tools.javac.api=ALL-UNNAMED \
                      --add-exports jdk.compiler/com.sun.tools.javac.file=ALL-UNNAMED \
                      --add-exports jdk.compiler/com.sun.tools.javac.parser=ALL-UNNAMED \
                      --add-exports jdk.compiler/com.sun.tools.javac.tree=ALL-UNNAMED \
                      --add-exports jdk.compiler/com.sun.tools.javac.util=ALL-UNNAMED \
                      --add-exports jdk.compiler/com.sun.tools.javac.code=ALL-UNNAMED \
                      --add-exports jdk.compiler/com.sun.tools.javac.comp=ALL-UNNAMED \
                      --add-exports jdk.compiler/com.sun.tools.javac.main=ALL-UNNAMED \
                      --add-exports jdk.compiler/com.sun.tools.javac.processing=ALL-UNNAMED"
    }

    triggers {
        pollSCM('* * * * *')
    }

    stages {
        stage('git repo') {
            steps {
                git url: 'https://github.com/Sravathi569/spring-petclinic.git', branch: 'main'
                    
            }
        }

        stage('java build and scan') {
            steps {
                withCredentials([string(credentialsId: 'sonarqube_id', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('sonar') {
                        sh """
                             mvn package sonar:sonar \
                            -Dsonar.organization=sravanthi569 \
                            -Dsonar.projectKey=Sravathi569_spring-petclinic \
                            -Dsonar.host.url=https://sonarcloud.io \
                            -Dsonar.login=$SONAR_TOKEN
                        """
                    }
                }
            }
        }

        stage('Upload to JFrog Artifactory') {
            steps {
                script {
                    def server = Artifactory.server(env.SERVER_ID)
                    def buildInfo = Artifactory.newBuildInfo()
                    server.upload(
                        spec: """{
                            "files": [{
                                "pattern": "target/*.jar",
                                "target": ""target": "libs-release-local/"
                            }]
                        }""",
                        buildInfo: buildInfo
                    )
                    server.publishBuildInfo(buildInfo)
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/target/*.jar'
            junit '**/target/surefire-reports/*.xml'
        }
        success {
            echo 'this pipeline good'
        }
        failure {
            echo 'this is waste pipeline'
        }
    }
}
