pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE_NAME = "balajikolle/openmrs-core"
        DOCKER_REGISTRY_CREDENTIALS = credentials('dockercredentials')
        NEW_VERSION = "env.BUILD_NUMBER"
        CHART_FILE =  "${WORKSPACE}/OpenmrsCore/Chart.yaml"
        // AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY')
        // AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = 'us-east-1' // Update with your desired region
        EKS_CLUSTER_NAME = 'eks-cluster'
        HELM_CHART_NAME = 'openmrs-core'
        NAMESPACE = 'develop'
        MYSQL_PASSWORD = "openmrs"
        MYSQL_ROOT_PASSWORD = "openmrs"
        OMRS_ADMIN_USER_PASSWORD = "Admin123" 
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                credentialsId: 'git_creds',
                url: 'https://github.com/balajiazuredevops/openmrs-core.git'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    def buildNumber = ${BUILD_NUMBER}
                    // Build the Docker image
                    sh "docker build -t ${DOCKER_IMAGE_NAME}:${buildNumber} ."
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    def buildNumber = ${BUILD_NUMBER}
                    def registryCredentials = ${DOCKER_REGISTRY_CREDENTIALS}
                    // def dockerRegistryUrl = env.DOCKER_REGISTRY_URL
                    def dockerImageTag = "${DOCKER_IMAGE_NAME}:${buildNumber}"

                    docker.withRegistry('', registryCredentials) {
                        docker.image(dockerImageTag).push()
                    }
                }
            }   
        }
        stage('Checkout helm template') {
            steps {
                git branch: 'main',
                credentialsId: 'git_creds',
                url: 'https://github.com/balajiazuredevops/terraformforeks.git'
            }
        }
        stage('Setup') {
            steps {
                script {
                    sh "aws eks update-kubeconfig --region ${AWS_DEFAULT_REGION} --name ${EKS_CLUSTER_NAME}"
                }
            }
        }
        stage('Replace App Version') {
            steps {
                script {
                    def chartContent = readFile(file: "${CHART_FILE}")
                    def updatedChartContent = chartContent.replaceAll(/appVersion:\s+.*/, "appVersion: ${NEW_VERSION}")
                    writeFile(file: "${CHART_FILE}", text: updatedChartContent)
                    echo "App Version in Chart.yaml has been updated to ${NEW_VERSION}"
                }
            }
        }
        stage('Create or Use Existing Namespace') {
            steps {
                script {
                    def namespaceExists = sh(script: "kubectl get namespace ${NAMESPACE} --ignore-not-found", returnStatus: true)
                    if (namespaceExists != 0) {
                        echo "Namespace ${NAMESPACE} does not exist. Creating..."
                        sh "kubectl create namespace ${NAMESPACE}"
                    } else {
                        echo "Using existing namespace ${NAMESPACE}"
                    }
                }
            }
        }
        stage('Deploy Helm Chart') {
            steps {
                script {
                    sh "cd ${WORKSPACE}/openmrs/OpenmrsCore"
                    sh "helm upgrade --install ${HELM_CHART_NAME} ${WORKSPACE}/openmrs/openmrs-core --set openmrs.imagename=${DOCKER_IMAGE_NAME}, openmrs.imagetag=${buildNumber},mysql.MYSQL_PASSWORD=${MYSQL_PASSWORD},mysql.MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD},mysql.OMRS_ADMIN_USER_PASSWORD=${OMRS_ADMIN_USER_PASSWORD} --namespace ${NAMESPACE}"
                }
            }
        }  
    }
    
    // post {
    //     always {
    //         // Clean up by logging out from Docker Hub
    //         sh "docker logout"
    //         sh "aws eks update-kubeconfig --region ${AWS_DEFAULT_REGION} --name ${EKS_CLUSTER_NAME} --unset"
    //     }
    // }
}