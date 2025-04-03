pipeline {
    agent any

    environment {
        RESOURCE_GROUP = 'Demo_Pract'
        APP_NAME = 'Demo-backendcode'
        GIT_CREDENTIALS = 'GitHub-Credentials' // Jenkins Git credentials ID
    }

    triggers {
        pollSCM('*/2 * * * *') // Auto-trigger build every 2 minutes if new changes are detected
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/MiddlewareTalent/Jenkins-demo-backendcode.git',
                    credentialsId: env.GIT_CREDENTIALS
            }
        }

        stage('Build Spring Boot App') {
            steps {
                bat 'mvn clean install -Dmaven.repo.local=.m2/repository'
            }
        }

        stage('Deploy to Azure') {
            steps {
                withCredentials([string(credentialsId: 'AZURE_WEBAPP_PUBLISH_PROFILE', variable: 'PUBLISH_PROFILE')]) {
                    bat """
                        az webapp deployment source config-zip \
                        --resource-group ${env.RESOURCE_GROUP} \
                        --name ${env.APP_NAME} \
                        --src-path target/*.jar \
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
