pipeline {
    environment {
        IMAGE_NAME = "webapp-jenkins"
        IMAGE_TAG = "v1"
        STAGING = "willahile-staging"
        PRODUCTION = "willahile-production"
    }
    agent none
    stages {
       stage('Build image') {
           agent any
           steps {
              script {
                sh 'docker build -t willahile/$IMAGE_NAME:$IMAGE_TAG .'
              }
           }
       }
       stage('Run container based on builded image') {
          agent any
          steps {
            script {
              sh '''
                  docker run --name $IMAGE_NAME -d -p 80:5000 -e PORT=5000 willahile/$IMAGE_NAME:$IMAGE_TAG
                  sleep 5
              '''
             }
          }
       }
       stage('Test image') {
           agent any
           steps {
              script {
                sh '''
                   curl http://172.17.0.1 | echo "Hello world!"
                '''
              }
           }
       }
       stage('Clean container') {
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
      stage('Push image in staging and deploy it') {
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
     stage('Push image in production and deploy it') {
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
}
