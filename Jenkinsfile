
pipeline {
    agent any
    tools {
        nodejs "node"
    }

    environment {
        BUCKET_NAME = "bucket-name-${params.Environment}"
    }

    stages {
        stage('Cloning Git') {
            steps {
                checkout scmGit(branches: [[name: "*/${Branch}"]], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Jainsi1/cloudfront-with-Jenkins.git']])
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Approval') {
            when {
                expression { 
                    params.Branch in ['prod','UAT'] 
                }
            }

            steps {
                script {
                  timeout(time: 5, unit: "MINUTES") {
                    input(
                        message: 'Do you want to approve the deployment?',
                        submitter: 'jainsi'
                    )
                   }
                }
            }
        }

        stage('Push to S3') {
            steps {
                    echo "Initiating deployment"
                    sh 'aws s3 sync build/ s3://${BUCKET_NAME}'
            }
        }

        stage('Deploy') {
            steps {
               script{
                   cmd = "aws cloudfront list-distributions | jq -r '.DistributionList.Items[] | [.Id, .Status, .Aliases.Items[0]] | @tsv'"
                   def list = sh(script: cmd, returnStdout: true)
                   echo list 

                   writeFile file: 'cloudfront_list.txt', text: list 
                   if(params.subdomain){
                        // get distribution id
                        get_cmd = """grep ${params.subdomain} 'cloudfront_list.txt' | awk '{print \$1}'"""
                        DISTRIBUTION_ID = sh(script: get_cmd, returnStdout: true)
                        println "Distribution ID = $DISTRIBUTION_ID"

                        // create invalidation id
                        create_cmd = """aws cloudfront create-invalidation --paths "/*" --distribution-id $DISTRIBUTION_ID """ 
                        INVALIDATION = sh(script: create_cmd, returnStdout: true)
                        println "Invalidation created as = $INVALIDATION"
	                }
                }//script 
            }//steps

        }
    }
}
 
