pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                credentialsId: 'fd28fdf5-53ab-45a7-a116-afc3494bc4c2',
                url: 'https://github.com/balajiazuredevops/terraformforeks.git'
            }
        }
        stage('Bild EKS'){
            steps {

                sh '''#!/usr/bin/env bash
                    cd ${WORKSPACE}/Terraform/
                    mkdir outplans
                    terraform init 
                    terraform validate
                    terraform plan  -out ${JOB_NAME}-${BUILD_NUMBER}
                    terraform show -no-color ${JOB_NAME}-${BUILD_NUMBER} > outplans/${JOB_NAME}-${BUILD_NUMBER}.txt
                    terraform apply -auto-approve ${JOB_NAME}-${BUILD_NUMBER} 
                    #-var access_key='${AWS_ACCESS_KEY_ID}' -var secret_key '{AWS_SECRET_ACCESS_KEY}' ${JOB_NAME}-${BUILD_NUMBER}
                '''

            }
        }
 
    }
    // post {
    //     always {
    //         archiveArtifacts artifacts: '${WORKSPACE}/Terraform/outplans/${JOB_NAME}-${BUILD_NUMBER}.txt'
    //     }
    // }
}