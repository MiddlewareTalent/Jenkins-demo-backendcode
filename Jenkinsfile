pipeline {
    agent any

    environment {
        RESOURCE_GROUP = 'Demo_Pract'
        APP_NAME = 'Demo-backendcode'
        GIT_CREDENTIALS = 'Git' // Jenkins Git credentials ID
        BRANCH = "${env.GIT_BRANCH ?: 'main'}"
        TRIVY_OUTPUT = 'trivy_report.json'
    }

    triggers {
        pollSCM('*/2 * * * *') // Auto-trigger build every 2 minutes if new changes are detected
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: BRANCH,
                    url: 'https://github.com/MiddlewareTalent/Jenkins-demo-backendcode.git',
                    credentialsId: env.GIT_CREDENTIALS
            }
        }

        stage('Security Scan with Trivy') {
            steps {
                sh 'trivy fs . --format json > ${TRIVY_OUTPUT}'
                archiveArtifacts artifacts: TRIVY_OUTPUT, fingerprint: true
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
                    if not exist deploy mkdir deploy
                    del /Q deploy\*
                    copy target\*.jar deploy\app.jar
                    tar -cvf deploy.zip deploy\*
                '''
            }
        }

        stage('Deploy to Azure') {
            steps {
                withCredentials([string(credentialsId: 'AZURE_WEBAPP_PUBLISH_PROFILE', variable: 'PUBLISH_PROFILE')]) {
                    bat """
                        az webapp deployment source config-zip ^
                        --resource-group ${env.RESOURCE_GROUP} ^
                        --name ${env.APP_NAME} ^
                        --src deploy.zip
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment to Azure App Service succeeded!'
            mail to: 'your-email@example.com',
                 subject: 'Jenkins Build Success',
                 body: "Build succeeded for branch ${BRANCH}."
        }
        failure {
            echo '❌ Deployment to Azure App Service failed!'
            mail to: 'your-email@example.com',
                 subject: 'Jenkins Build Failed',
                 body: "Build failed for branch ${BRANCH}. Check logs for details."
        }
    }
}
