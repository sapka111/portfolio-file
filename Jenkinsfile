pipeline {
  agent any

  environment {
    AWS_CREDENTIALS = 'aws-deployer'       // your Jenkins IAM creds ID
    AWS_REGION      = 'us-east-2'
    S3_BUCKET       = 'portfoliowebsitefinal'
    CF_DIST_ID      = 'E3ABCDEF12345'      // your CloudFront ID
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Deploy to S3') {
   steps {
     withCredentials([[
       $class: 'AmazonWebServicesCredentialsBinding',
       credentialsId: env.AWS_CREDENTIALS
     ]]) {
-      sh """
-        aws configure set region $AWS_REGION
-        aws s3 sync site/ s3://$S3_BUCKET/ --delete --acl public-read
-      """

    stage('Invalidate CF') {
      steps {
        withCredentials([[
          $class: 'AmazonWebServicesCredentialsBinding',
          credentialsId: env.AWS_CREDENTIALS
        ]]) {
          sh """
            aws cloudfront create-invalidation \
              --distribution-id $CF_DIST_ID \
              --paths '/*'
          """
        }
      }
    }
  }
}
