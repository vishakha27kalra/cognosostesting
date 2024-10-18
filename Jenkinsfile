pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1' // Change to your region
        INSTANCE_ID = 'i-01d24802a35a8d0af' // Change to your instance ID
    }

    stages {
        stage('AWS Authentication') {
            steps {
                script {
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'official-aws',
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY',
                    ]]) {
                        // Fetch the volume ID
                        def volId = sh(script: "aws ec2 describe-volumes --filters Name=attachment.instance-id,Values=${env.INSTANCE_ID} --query 'Volumes[*].VolumeId' --output text", returnStdout: true).trim()
                        echo "Volume ID: ${volId}"

                        // Create a snapshot and capture the Snapshot ID
                        def snapshotId = sh(script: """
                            aws ec2 create-snapshot --region ${env.AWS_REGION} --volume-id ${volId} --output text --query 'SnapshotId'
                        """, returnStdout: true).trim()
                        
                        echo "Snapshot ID: ${snapshotId}"

                        // Update CloudFormation stack
                        sh """
                            aws cloudformation update-stack \
                                --stack-name ${stackName} \
                                --template-body file:///tmp/website2.yaml \
                                --parameters ParameterKey=SnapshotId,ParameterValue=${snapshotId} ParameterKey=VolumeId,ParameterValue=${volId} \
                                --region ${env.AWS_REGION} \
                                --capabilities CAPABILITY_IAM
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
        failure {
            echo 'Build failed. Please check the logs for details.'
        }
        success {
            echo 'Build succeeded!'
        }
    }
}

