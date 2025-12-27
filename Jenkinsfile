pipeline {
    agent {
        node {
            label 'AGENT-1'
        }
    }

    environment {
        REGION    = "us-east-1"
        ACC_ID    = "784585544641"
        PROJECT   = "roboshop"
        COMPONENT = "catalogue"
    }

    options {
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
    } 

    parameters {
        string(
            name: 'appVersion',
            description: 'Docker image version passed from CI'
        ) 
        choice(
            name: 'deploy_to',
            choices: ['dev', 'qa', 'prod'],
            description: 'Pick the environment'
        )
    } 

    stages {

        stage('Deploy') {
            steps {
                script {
                    echo "Deploying image version: ${params.appVersion} to environment: ${params.deploy_to}"

                    withAWS(credentials: 'aws-creds', region: REGION) {
                        sh """
                            aws eks update-kubeconfig --region=${REGION} --name=${PROJECT}-${params.deploy_to}
                            kubectl set image deployment/${COMPONENT} ${COMPONENT}=${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${PROJECT}/${COMPONENT}:${params.appVersion} -n ${PROJECT}
                            kubectl rollout status deployment/${COMPONENT} -n ${PROJECT} --timeout=120s
                        """
                    }
                }
            }
        }

        stage('Functional Testing') {
            when {
                expression { params.deploy_to == "dev" }
            }
            steps {
                script {
                    echo "Run functional test cases on DEV environment"
                }
            }
        }

        stage('Integration Testing') {
            when {
                expression { params.deploy_to == "dev" }
            }
            steps {
                script {
                    echo "Run integration test cases on DEV environment"
                }
            }
        }

    } 

    post { 
        always { 
            echo 'CD pipeline finished'
        }
        success {
            echo "Deployment of version ${params.appVersion} succeeded"
            deleteDir()
        }
        failure {
            echo "Deployment of version ${params.appVersion} failed"
        }
    }
}
