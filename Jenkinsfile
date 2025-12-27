pipeline {
   agent {
        node {
            label 'AGENT-1'
        }
    }
    environment {
        appVersion = ''
        REGION = "us-east-1"
        ACC_ID = "784585544641"
        PROJECT = "roboshop"
        COMPONENT = "catalogue"

    }
    options {
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
    } 
    parameters {
        string(name: 'appVersion',  description: 'image version of the application') 
        choice(name: 'deploy_to', choices: ['dev', 'qa', 'prod'], description: 'Pick the environment')
    } 
// build
    stages {
        stage('deploy') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                         sh """  
                             aws eks update-kubeconfig --region $REGION --name "$PROJECT-${params.deploy_to}" 
                             kubectl get nodes
                             kubectl apply -f 01-namespace.yaml
                             sed -i "s/IMAGE_VERSION/${params.appVersion}/g" values-${params.deploy_to}.yaml
                             helm upgrade --install $COMPONENT -f values-${params.deploy_to}.yaml -n $PROJECT .
                       """
                    }
                }
            }
        }
        stage ('check status') {
            steps{
                script{
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                        def deploymentstatus = sh(returnStdout: true, script: "kubectl rollout status deployment/catalogue --timeout=30s -n $PROJECT  || echo failed").trim()
                        if (deploymentstatus.contains("successfully rolled out")) {
                            echo "deployment is success"
                        } else {
                            sh """
                                helm rollback $COMPONENT -n $PROJECT
                                sleep 20
                            """
                            def rollbackstatus = sh(returnStdout: true, script: "kubectl rollout status deployment/catalogue --timeout=30s -n $PROJECT || echo failed").trim()
                            if (deploymentstatus.contains("successfully rolled out")) {
                                error  ("deployment is failure, rollback is success")
                            }
                            else {
                                echo "deployment is failure, rollback is failure.Application is not running"
                            }
                        }
                    }
                }
            }
        }
        stage('Functional Testing'){
            when {
                expression { params.deploy_to = "dev" }
            }
            steps{
                script{
                    echo "Run functional test cases"

                }
            }
        }
         stage('Integration Testing'){
            when {
                expression { params.deploy_to = "dev" }
            }
            steps{
                script{
                    echo "Run integrational test cases"

                }
            }
        }
    
    }
    post { 
        always { 
            echo 'I will always say Hello again!'
        }
        success {
            echo 'hi this is success'
            deleteDir()
        }
        failure {
            echo 'hi, this is failure'
        }
    }
}
    
