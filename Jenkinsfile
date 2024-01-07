pipeline {
    agent any

    stages {
        stage('Delete the workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Installing Java and Maven') {
            steps {
                script {
                    def maven_exists = fileExists('/usr/share/maven')
                    if (maven_exists == true) {
                        echo 'Skipping Maven install - already exists'
                    } else {
                        echo 12345 | sudo -S apt-get update -y
                        sh 'sudo apt install -y wget tree unzip openjdk-11-jdk maven'
                    }
                }
            }
        }
        stage('Download Java Code') {
            steps {
                git branch: 'main', credentialsId: 'git-repo-creds', url: 'git@github.com:Shavejansari/java-jenkins-repo.git'
            }
        }
        stage('Compiling and Running Test Cases') {
            steps {
                sh 'mvn clean'
                sh 'mvn compile'
                sh 'mvn test'
            }
        }
        stage('Creating Package') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Deploying Application') {
            steps {
                script {
                    withEnv(['JENKINS_NODE_COOKIE=dontkill']) {
                        sh """
                            nohup java -jar ./target/jenkin-java-training-0.0.1-SNAPSHOT.jar &
                        """
                    }
                }
            }
        }
        stage('Send Slack Notification') {
            steps {
                slackSend color: 'warning', message: """
                    Mr. Deeds: Please approve ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.JOB_URL} | Open>)
                """
            }
        }
        stage('Request Input') {
            steps {
                input 'Please approve or deny this build'
            }
        }
    }
    post {
        success {
            slackSend color: 'warning', message: """
                Build ${env.JOB_NAME} ${env.BUILD_NUMBER} was successful! :)
            """
        }
        failure {
            slackSend color: 'warning', message: """
                Build ${env.JOB_NAME} ${env.BUILD_NUMBER} failed :(
            """
        }
    }
}
