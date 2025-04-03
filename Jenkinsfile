pipeline {
    agent any

    environment {
        RESOURCE_GROUP = 'Demo_Pract'
        APP_NAME = 'Demo-backendcode'
        GIT_CREDENTIALS = 'Git' // Jenkins Git credentials ID
    }

    triggers {
        pollSCM('H/2 * * * *') // Poll SCM every 2 minutes for changes
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "*/main"]],
                        userRemoteConfigs: [[
                            url: 'https://github.com/MiddlewareTalent/Jenkins-demo-backendcode.git',
                            credentialsId: env.GIT_CREDENTIALS
                        ]]
                    ])
                }
                script {
                    bat 'git log -1 --pretty=format:"Latest Commit: %h - %an: %s"'
                }
            }
        }

        stage('Build Spring Boot App') {
            steps {
                bat 'mvn clean package -Dmaven.repo.local=.m2/repository'
            }
        }

        stage('Prepare Deployment Package') {
            steps {
                bat '''
                    echo Cleaning previous deployment artifacts...
                    if not exist deploy mkdir deploy
                    del /Q deploy\\*
                    copy target\\*.jar deploy\\app.jar
                    tar -cvf deploy.zip deploy
                '''
            }
        }

        stage('Deploy to Azure') {
            steps {
                withCredentials([string(credentialsId: 'AZURE_WEBAPP_PUBLISH_PROFILE', variable: 'PUBLISH_PROFILE')]) {
                    bat '''
                        echo Deploying to Azure Web App...
                        az webapp deployment source config-zip ^
                        --resource-group "%RESOURCE_GROUP%" ^
                        --name "%APP_NAME%" ^
                        --src deploy.zip
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment to Azure App Service succeeded! 🎉'
        }
        failure {
            echo '❌ Deployment to Azure App Service failed. Check logs for details.'
        }
    }
}
