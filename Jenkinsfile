pipeline {
    agent any
    tools {
        terraform 'terraform-latest' // Must match the name you set in Manage Jenkins -> Tools
    }
    
    parameters {
        choice(name: 'ACTION', choices: ['apply', 'destroy'], description: 'Choose whether to apply or destroy the Terraform EKS module')
    }

    environment {
        TF_DIR = 'eks-install'
        AWS_DEFAULT_REGION = 'ap-south-1'
        
        // This pulls the two "Secret text" credentials you already created in Jenkins
        // using the exact IDs shown in your screenshot ('accesskey' and 'secretaccesskey')
        AWS_ACCESS_KEY_ID = credentials('accesskey')
        AWS_SECRET_ACCESS_KEY = credentials('secretaccesskey')
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Init') {
            steps {
                dir("${env.TF_DIR}") {
                    sh 'terraform init'
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                dir("${env.TF_DIR}") {
                    sh "terraform plan ${params.ACTION == 'destroy' ? '-destroy' : ''} -out=tfplan"
                }
            }
        }

        stage('Terraform Execute') {
            steps {
                dir("${env.TF_DIR}") {
                    script {
                        if (params.ACTION == 'apply') {
                            sh 'terraform apply -auto-approve tfplan'
                        } else if (params.ACTION == 'destroy') {
                            sh 'terraform apply -destroy -auto-approve'
                        }
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs() // Cleans up workspace after execution
        }
    }
}
