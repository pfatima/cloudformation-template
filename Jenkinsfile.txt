pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'ap-south-1' 
        STACK_NAME = 's3-bucket-stack'
        TEMPLATE_PATH = "s3-bucket.yaml"
    }

    stages {
        stage('Clone Git Repo') {
            steps {
                git url: 'https://github.com/pfatima/Aws-Solution-Architect.git', branch: 'main'
            }
        }

        stage('Deploy CloudFormation Stack') {
            steps {
                script {
                    // Check if file exists
                    if (!fileExists("${TEMPLATE_PATH}")) {
                        error "CloudFormation template not found at: ${TEMPLATE_PATH}"
                    }

                    // Run deployment command
                    sh """
                        aws cloudformation deploy \
                          --template-file "${TEMPLATE_PATH}" \
                          --stack-name "${STACK_NAME}" \
                          --capabilities CAPABILITY_NAMED_IAM
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ CloudFormation stack deployed successfully."
        }
        failure {
            echo "❌ CloudFormation stack deployment failed."
        }
    }
}
