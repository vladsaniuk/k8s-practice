pipeline {
    agent any
    environment {
        TIMESTAMP = sh (
        script: 'date +%s',
        returnStdout: true
        ).trim()
        APP_VERSION = "${TIMESTAMP}-${BUILD_ID}"
    }

    stages {
        stage('Build app') {
            when {
                expression {
                    BRANCH_NAME == 'master'
                }
            }
            steps {
                    sh './gradlew build'
            }
        }

        stage('Test app') {
            steps {
                sh './gradlew test'
            }

            post {
                failure {
                    sh 'exit 1'
                }
            }
        }

        stage('Login to registry') {
            when {
                expression {
                    BRANCH_NAME == 'master'
                }
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                    }
                }
            }
        }

        stage('Build image') {
            when {
                expression {
                    BRANCH_NAME == 'master'
                }
            }
            steps {
                script {
                    sh "docker build -t vladsanyuk/java-app:${APP_VERSION} ."
                }
            }
        }

        stage('Push to registry') {
            when {
                expression {
                    BRANCH_NAME == 'master'
                }
            }
            steps {
                script {
                    sh "docker push vladsanyuk/java-app:${APP_VERSION}"
                }
            }
        }
    }
}