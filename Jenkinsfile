pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'stg', 'prod'],
            description: 'Select the environment workspace to deploy to'
        )
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Select Workspace') {
            steps {
                script {
                    sh "terraform workspace select ${params.ENVIRONMENT} || terraform workspace new ${params.ENVIRONMENT}"
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                sh "terraform plan -out=tfplan -var-file=${params.ENVIRONMENT}.tfvars"
            }
        }

        stage('Approval') {
            steps {
                timeout(time: 30, unit: 'MINUTES') {
                    input message: "Do you want to apply changes to the '${params.ENVIRONMENT}' environment?",
                          ok: "Approve"
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                sh 'terraform apply tfplan'
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline completed successfully for environment: ${params.ENVIRONMENT}!"
        }
        failure {
            echo "❌ Pipeline failed during execution in environment: ${params.ENVIRONMENT}."
        }
        always {
            sh 'rm -f tfplan'
        }
    }
}
