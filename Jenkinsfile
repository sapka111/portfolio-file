pipeline {
  agent any

  environment {
    AWS_CREDENTIALS = 'aws-deployer'       // your Jenkins AWS creds ID
    AWS_REGION      = 'us-east-2'          // match your bucket’s region
    S3_BUCKET       = 'portfoliowebsitefinal'
    CF_DIST_ID      = 'E3ABCDEF12345'      // your CloudFront distribution ID
  }
  stages {
   stage('Ensure S3 Bucket') {
      steps {
        withCredentials([[
          $class: 'AmazonWebServicesCredentialsBinding',
          credentialsId: env.AWS_CREDENTIALS
        ]]) {
          sh '''
            # Try to create the bucket; ignore “already exists” errors
            aws s3 mb s3://$S3_BUCKET --region $AWS_REGION || true
          '''
        }
      }
    }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Deploy to S3') {
      steps {
        withCredentials([[
          $class: 'AmazonWebServicesCredentialsBinding',
          credentialsId: env.AWS_CREDENTIALS
        ]]) {
          // use single-quotes so Groovy doesn’t try to interpolate
          sh '''
            aws configure set region $AWS_REGION
            aws s3 sync . s3://$S3_BUCKET/ --delete --acl public-read \
              --exclude ".git/*" --exclude "Jenkinsfile"
          '''
        }
      }
    }

    stage('Invalidate CloudFront Cache') {
      steps {
        withCredentials([[
          $class: 'AmazonWebServicesCredentialsBinding',
          credentialsId: env.AWS_CREDENTIALS
        ]]) {
          sh '''
            aws cloudfront create-invalidation \
              --distribution-id $CF_DIST_ID \
              --paths "/*"
          '''
        }
      }
    }
  }

  post {
    success { echo "✅ Deployment complete!" }
    failure { echo "❌ Deployment failed." }
  }
}
