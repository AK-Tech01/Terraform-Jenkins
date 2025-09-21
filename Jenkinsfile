pipeline {
    agent any

    parameters {
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/AK-Tech01/Terraform-Jenkins.git'
            }
        }

        stage('Terraform Init & Plan') {
            steps {
                withAWS(credentials: 'AWS-Jenkins-User', region: 'aus-east-1') {
                    dir('terraform') {
                        sh '''
                            terraform init -input=false
                            terraform plan -out=tfplan -input=false
                            terraform show -no-color tfplan > tfplan.txt
                        '''
                    }
                }
            }
        }

        stage('Approval') {
            when {
                not { equals expected: true, actual: params.autoApprove }
            }
            steps {
                script {
                    def plan = readFile 'terraform/tfplan.txt'
                    input message: "Do you want to apply the plan?",
                          parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                withAWS(credentials: 'aws-jenkins-user', region: 'ap-south-1') {
                    dir('terraform') {
                        sh 'terraform apply -input=false tfplan'
                    }
                }
            }
        }
    }
}
