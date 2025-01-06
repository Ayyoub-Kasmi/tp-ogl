pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                echo 'Running unit tests...'
                script {
                    try {
                        sh './gradlew test'
                        junit '**/build/test-results/test/*.xml'
                    } catch (Exception e) {
                        echo "Test stage failed: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        error("Test stage failed")
                    }
                }
            }
        }

        stage('Code Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                script {
                    try {
//                         sh "./gradlew sonarqube -Dsonar.host.url=${SONAR_HOST_URL}"
                        sh "./gradlew sonar"
                    } catch (Exception e) {
                        echo "SonarQube analysis failed: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        error("SonarQube analysis failed")
                    }
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building the project...'
                script {
                    try {
                        sh './gradlew build'
                        archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true
                    } catch (Exception e) {
                        echo "Build stage failed: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        error("Build stage failed")
                    }
                }
            }
        }

        stage('Deployy') {
            steps {
                echo 'Deploying to MyMavenRepo...'
                sh "./gradlew publish"
            }
        }

        stage('Send Notification') {
            steps {
                script {
                    def result = currentBuild.result ?: 'SUCCESS'
                    echo result
                    if (result == 'SUCCESS') {
                        mail to: 'la_kasmi@esi.dz',
                             subject: "Jenkins Build #${env.BUILD_NUMBER} Success",
                             body: "The build #${env.BUILD_NUMBER} has been successfully completed.\n\nFollow this link for more details: ${env.BUILD_URL}"
                    } else {
                        mail to: 'la_kasmi@esi.dz',
                             subject: "Jenkins Build #${env.BUILD_NUMBER} Failure",
                             body: "The build #${env.BUILD_NUMBER} has failed.\n\nFollow this link for more details: ${env.BUILD_URL}"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution finished.'
        }

        success {
            echo 'Pipeline succeeded!'
            slackSend color: "good", message: "Message from Jenkins Pipeline: SUCCESS"
            mail to: 'la_kasmi@esi.dz',
                 subject: "Jenkins Build #${env.BUILD_NUMBER} Success",
                 body: "The build #${env.BUILD_NUMBER} has been successfully completed.\n\nFollow this link for more details: ${env.BUILD_URL}"

        }

        failure {
            echo 'Pipeline failed!'
            slackSend color: "danger", message: "Message from Jenkins Pipeline: FAIL"
            mail to: 'la_kasmi@esi.dz',
                 subject: "Jenkins Build #${env.BUILD_NUMBER} Failure",
                 body: "The build #${env.BUILD_NUMBER} has failed.\n\nFollow this link for more details: ${env.BUILD_URL}"
        }
    }
}
