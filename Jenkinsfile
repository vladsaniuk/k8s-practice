pipeline {
    agent any
    parameters {
      string(name: "CLUSTER_NAME", defaultValue: "my-eks-cluster", description: "EKS cluster name")
    }
    environment {
        TIMESTAMP = sh (
        script: 'date +%s',
        returnStdout: true
        ).trim()
        APP_VERSION = "${TIMESTAMP}-${BUILD_ID}"
        AWS_ACCESS_KEY_ID = credentials('aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('aws_secret_access_key')
    }

    stages {
        stage('Build app') {
            when {
                expression {
                    BRANCH_NAME == 'master'
                }
            }
            steps {
                    echo "Building app"
                    sh './gradlew build'
            }
        }

        stage('Test app') {
            steps {
                echo "Running tests"
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
                    echo "Logging into Docker Hub"
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
                    echo "Building container image"
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
                    echo "Pushing image to registry"
                    sh "docker push vladsanyuk/java-app:${APP_VERSION}"
                    echo "Clean-up image"
                    sh "docker rmi vladsanyuk/java-app:${APP_VERSION}"
                }
            }
        }

        stage('Get kubeconfig') {
            when {
                expression {
                    BRANCH_NAME == 'master'
                }
            }
            steps {
                script {
                    echo "Getting kubeconfig"
                    sh "aws eks update-kubeconfig --region us-east-1 --name ${params.CLUSTER_NAME}"
                }
            }
        }

        stage('Deploy to cluster') {
            when {
                expression {
                    BRANCH_NAME == 'master'
                }
            }
            steps {
                script {
                    echo "Deploying app"
                    sh "envsubst < helm/my-app/values-my-app.yaml | helm upgrade -f helm/my-app/values-my-app.yaml my-app-release ./helm/my-app/my-app -n my-app --install --create-namespace"
                    echo "Deploying DB"
                    sh "helm upgrade -f helm/mysql/values-mysql-bitnami.yaml mysql-release bitnami/mysql -n my-app --install --create-namespace"
                }
            }
        }

        post {
        // Clean after build
            always {
                cleanWs(cleanWhenNotBuilt: false,
                        deleteDirs: true,
                        cleanWhenAborted: true,
                        cleanWhenFailure: true,
                        cleanWhenSuccess: true,
                        cleanWhenUnstable: true)
            }
        }
    }
}