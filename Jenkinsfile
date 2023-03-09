/* import shared library */
@Library('Bah27-share-library')_

pipeline {
    environment {
        IMAGE_NAME = "applicationweb"
        IMAGE_TAG = "latest"
        STAGING = "manma27-staging"
        PRODUCTION = "manma27-production"
    }
    agent none
    stages {
       stage('Construction de l\'application') {
           agent any
           steps {
              script {
                sh 'docker build -t manma27/$IMAGE_NAME:$IMAGE_TAG .'
              }
           }
       }
       stage('Lancement du conteneur en se basant sur l\'image') {
          agent any
          steps {
            script {
              sh '''
                  docker run --name $IMAGE_NAME -d -p 80:5000 -e PORT=5000 manma27/$IMAGE_NAME:$IMAGE_TAG
                  sleep 5
              '''
             }
          }
       }
       stage('Teste de l\'image') {
           agent any
           steps {
              script {
                sh '''
                   curl localhost | echo "Hello world!"
                '''
              }
           }
       }
       stage('Suppression de l\'image') {
          agent any
          steps {
             script {
               sh '''
                   docker stop $IMAGE_NAME
                   docker rm $IMAGE_NAME
               '''
             }
          }
      }
      stage('Téléchargement de l\'image dans l\'environnement de staging puis le déployer') {
        when {
            expression { GIT_BRANCH == 'origin/main' }
        }
        agent any
        environment {
            HEROKU_API_KEY = credentials('heroku_api_key')
        }
        steps {
           script {
             sh '''
                heroku container:login
                heroku create $STAGING || echo "projets already exist"
                heroku container:push -a $STAGING web
                heroku container:release -a $STAGING web
             '''
           }
        }
     }
     stage('Téléchargement et déploiement en production') {
       when {
           expression { GIT_BRANCH == 'origin/main' }
       }
       agent any
       environment {
           HEROKU_API_KEY = credentials('heroku_api_key')
       }
       steps {
          script {
            sh '''
               heroku container:login
               heroku create $PRODUCTION || echo "projets already exist"
               heroku container:push -a $PRODUCTION web
               heroku container:release -a $PRODUCTION web
            '''
          }
       }
     }
  }
  post {
     always {
       script {
         slackNotifier currentBuild.result
     }
    }
  }
}
