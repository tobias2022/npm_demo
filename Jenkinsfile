pipeline {
   agent any
   environment {
      CI = 'true'
   }
   stages {
         stage('Install Packages') {
            steps {
            sh 'npm install'
            }
         }
         stage('Test and Build') {
            parallel {
            stage('Run Tests') {
               steps {
                  sh 'npm run test'
               }
            }
            stage('Create Build Artifacts') {
               steps {
                  sh 'npm run build'
               }
            }
            }
         }
          stage('Validate CF syntax') {
               steps {
                   withAWS(region:'ap-southeast-2', credentials:'aws-credentials') {
                  sh 'aws cloudformation validate-template --template-body file://s3.yml'
                   }
               }
            }
             stage('S3 Bucket') {
               steps {
                  withAWS(region:'ap-southeast-2', credentials:'aws-credentials') {
                           sh 'aws cloudformation deploy \
                        --template-file git-practice.yml \
                        --stack-name git-practice-stack \
                        --parameter-overrides BucketName=tobias-git-practice'
                  }
               }
            }
         stage('Deploy to Production') {
               steps {
               script {
                  if (env.BRANCH_NAME == 'main') {
                  input message: 'Deploy to production? (Click "Proceed" to continue)'
                  withAWS(region:'ap-southeast-2', credentials:'aws-credentials') {
                        s3Delete(bucket: 'tobias-git-practice', path:'**/*')
                        s3Upload(bucket: 'tobias-git-practice', workingDir:'build', includePathPattern:'**/*')
                  }
                     } else {
                  echo 'I execute elsewhere'
                  }
               }
               }
         }
   }
}