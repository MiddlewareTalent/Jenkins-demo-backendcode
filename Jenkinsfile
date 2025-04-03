pipeline {
    agent any

    environment {
        RESOURCE_GROUP = 'Demo_Pract'  //resorce group name
        APP_NAME = 'Demo-backendcode'
        GIT_CREDENTIALS = 'Git' // Jenkins GitHub Credentials
    }

    triggers {
        pollSCM('* * * * *') // Poll every minute (adjust as needed)
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/MiddlewareTalent/Jenkins-demo-backendcode.git',
                    credentialsId: env.GIT_CREDENTIALS
            }
        }

        stage('Set Up JDK 17') {
            steps {
                bat 'choco install openjdk17'
                bat 'setx JAVA_HOME "C:\\Program Files\\OpenJDK\\openjdk-17"'
            }
        }

        stage('Build Spring Boot App') {
            steps {
                bat 'mvn clean install'
            }
        }

        stage('Deploy to Azure') {
            steps {
                withCredentials([string(credentialsId: 'AZURE_PUBLISH_PROFILE', variable: 'PUBLISH_PROFILE')]) {
                    bat """
                        az webapp deploy \
                        --resource-group ${env.RESOURCE_GROUP} \
                        --name ${env.APP_NAME} \
                        --src-path target/*.jar \
                        --type jar \
                        --publish-profile "${PUBLISH_PROFILE}"
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment to Azure App Service succeeded!'
        }
        failure {
            echo '❌ Deployment to Azure App Service failed!'
        }
    }
}
