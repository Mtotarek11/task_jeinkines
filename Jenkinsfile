pipeline {
    agent any

    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['dev', 'prod'],
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

        stage('Approval & Notification') {
            steps {
                script {
                    // بيجيب لينك الـ Console Output الحالي
                    def consoleUrl = "${BUILD_URL}console"
                    
                    // بيعت إيميل للمسؤول إن فيه Pipeline مستنية موافقة
                    emailext (
                        subject: "⏳ Action Required: Terraform Plan for [${params.ENVIRONMENT}] waiting for approval",
                        body: """
                        Hello,\n\nA new Terraform deployment for environment <b>${params.ENVIRONMENT}</b> requires your approval.\n\n
                        You can review the plan output and approve it here:\n
                        <a href="${consoleUrl}">Click here to view Console Output & Approve</a>\n\n
                        Build Name: ${JOB_NAME} #${BUILD_NUMBER}
                        """,
                        to: "your-email@example.com", // حط إيميلك هنا أو اسحبه ديناميك
                        mimeType: 'text/html'
                    )
                }

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
            script {
                def consoleUrl = "${BUILD_URL}console"
                emailext (
                    subject: "✅ SUCCESS: Pipeline ${JOB_NAME} [#${BUILD_NUMBER}] - ${params.ENVIRONMENT}",
                    body: "The pipeline for environment <b>${params.ENVIRONMENT}</b> completed successfully.<br><br>View logs here: <a href='${consoleUrl}'>Console Output</a>",
                    to: "your-email@example.com",
                    mimeType: 'text/html'
                )
            }
        }
        failure {
            script {
                def consoleUrl = "${BUILD_URL}console"
                emailext (
                    subject: "❌ FAILED: Pipeline ${JOB_NAME} [#${BUILD_NUMBER}] - ${params.ENVIRONMENT}",
                    body: "The pipeline for environment <b>${params.ENVIRONMENT}</b> has FAILED.<br><br>Check logs here: <a href='${consoleUrl}'>Console Output</a>",
                    to: "your-email@example.com",
                    mimeType: 'text/html'
                )
            }
        }
        always {
            sh 'rm -f tfplan'
        }
    }
}
