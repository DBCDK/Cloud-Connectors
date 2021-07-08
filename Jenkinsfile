#!groovy

def workerNode = "devel10"

pipeline {
    agent {
        label workerNode
    }
    options {
        timestamps()
    }
    tools {
        maven "Maven 3"
    }
    stages {
        stage("clear workspace") {
            steps {
                deleteDir()
                checkout scm
            }
        }
        stage("build") {
			steps {
                script {
                    def status = sh returnStatus: true, script: """
                        mvn -B -Dmaven.repo.local=\$WORKSPACE/.repo -pl Kafka/KafkaRAR -am verify
                    """
                }
            }
        }
        stage("deploy") {
            when {
                branch "dbc-pipeline"
            }
            steps {
                sh "mvn -pl Kafka/KafkaRAR -am jar:jar deploy:deploy"
            }
        }
    }
    post {
        failure {
            script {
                if ("${env.BRANCH_NAME}".equals('master')) {
                    slackSend(channel: "meta-notifications",
                            color: 'warning',
                            message: "${env.JOB_NAME} #${env.BUILD_NUMBER} failed and needs attention: ${env.BUILD_URL}",
                            tokenCredentialId: 'slack-global-integration-token')
                }
            }
        }
    }
}
