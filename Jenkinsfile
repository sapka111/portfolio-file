pipeline {
    agent any

    environment {
        AWS_CREDENTIALS = 'aws-deployer'           
        AWS_REGION      = 'us-east-2'              // your bucket’s region
        S3_BUCKET       = 'portfoliofinalbucket'   // <— updated!
        CF_DIST_ID      = 'E2W41DBKCS67ZQ'          
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Deploy to S3') {
  steps {
    withCredentials([[
      $class: 'AmazonWebServicesCredentialsBinding',
      credentialsId: "${AWS_CREDENTIALS}"
    ]]) {
      sh '''
        aws configure set region ${AWS_REGION}
        aws s3 sync . s3://${S3_BUCKET}/ --delete \
          --exclude ".git/*" --exclude "Jenkinsfile"
      '''
    }
  }
}

        stage('Invalidate CloudFront') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: "${AWS_CREDENTIALS}"
                ]]) {
                    sh '''
                      aws cloudfront create-invalidation \
                        --distribution-id ${CF_DIST_ID} \
                        --paths "/*"
                    '''
                }
            }
        }
    }

    post {
        success { echo '✅ Deployment succeeded!' }
        failure { echo '❌ Deployment failed.' }
    }
}
