pipeline {
    agent any
    environment {
        IMAGE_VERSION = "${env.BUILD_NUMBER}"
        LAST_CLOUD_RUN_REVISION = "myapplication-${currentBuild.previousBuild.getNumber()}"
        IMAGE_NAME = "gcr.io/<GOOGLE_CLOUD_PROJECT_ID>/myapplication:v1.1.${IMAGE_VERSION}"
        PREVIOUS_IMAGE_NAME = "gcr.io/<REPLACE_GOOGLE_CLOUD_PROJECT_ID>/myapplication:v1.1.${currentBuild.previousBuild.getNumber()}"
    }
    stages {
        stage('Git checkout') {
            steps{
                git (
                        credentialsId: '<JENKINS_CREDENTIALS_ID>',
                        url: '<GIT_URL>',
                        branch : 'develop'
                )
            }
        }
        stage("Build new docker image"){
            steps{
                sh "docker build --tag=${IMAGE_NAME} . --file=docker/Dockerfile"
            }
        }
        stage("Push to Google Container Registry"){
            steps{
                sh "docker push ${IMAGE_NAME}"
            }
        }
        stage("Deploy new image to Cloud Run"){
            steps{
                sh "gcloud run deploy myapplication --image  ${IMAGE_NAME} --platform=managed --region=us-central1 --port=8080 --revision-suffix=${IMAGE_VERSION}"
            }
        }
        stage("Delete last Cloud Run revision"){
            steps{
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    sh "yes | gcloud run revisions delete ${LAST_CLOUD_RUN_REVISION} --platform=managed --region=us-central1"
                }
            }
        }
    }
}
