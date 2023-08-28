pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE_NAME = "balajikolle/openmrs-core"
        // DOCKER_REGISTRY_CREDENTIALS = credentials('dockercredentials')
        DOCKER_HUB_USERNAME = "balajikolle"
        DOCKER_HUB_PASSWORD = credentials('docker-passwd')
        NEW_VERSION = "env.BUILD_NUMBER"
        CHART_FILE =  "${WORKSPACE}/OpenmrsCore/Chart.yaml"
        // AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY')
        // AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_DEFAULT_REGION = 'us-east-1' // Update with your desired region
        EKS_CLUSTER_NAME = 'eks-cluster'
        HELM_CHART_NAME = 'OpenmrsCore'
        //NAMESPACE = 'develop'
        MYSQL_PASSWORD = "openmrs"
        MYSQL_ROOT_PASSWORD = "openmrs"
        OMRS_ADMIN_USER_PASSWORD = "Admin123" 
    }
    parameters {
        choice(
            choices: 'develop\nreview\npre-prod\nproduction',
            description: 'Select the target namespace',
            name: 'TARGET_NAMESPACE'
        )
        string(
            defaultValue: 'OpenmrsCore',
            description: 'Name of the Helm chart',
            name: 'HELM_CHART_NAME'
        )
        // Add more dynamic parameters as needed
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master',
                credentialsId: 'git_credt',
                url: 'https://github.com/balajiazuredevops/openmrs-core.git'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    //def buildNumber = ${BUILD_NUMBER}
                    // Build the Docker image
                    sh "docker build -t ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} ."
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    //def buildNumber = ${BUILD_NUMBER}
                    // def registryCredentials = ${DOCKER_REGISTRY_CREDENTIALS}
                    // def dockerRegistryUrl = env.DOCKER_REGISTRY_URL
                    //def dockerImageTag = "${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}"

                    docker.withRegistry('', DOCKER_REGISTRY_CREDENTIALS) {
                        sh "docker push ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} "
                        
                    }
                }
            }   
        }
        stage('Checkout helm template') {
            steps {
                git branch: 'main',
                credentialsId: 'git_credt',
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
                    def helmChart = params.HELM_CHART_NAME
                    def targetNamespace = params.TARGET_NAMESPACE
                    def namespaceExists = sh(script: "kubectl get namespace ${targetNamespace} --ignore-not-found", returnStatus: true)
                    if (namespaceExists != 0) {
                        echo "Namespace ${targetNamespace} does not exist. Creating..."
                        sh "kubectl create namespace ${targetNamespace}"
                    } else {
                        echo "Using existing namespace ${targetNamespace}"
                    }
                }
            }
        }
        stage('Deploy Helm Chart') {
            steps {
                script {
                    def helmChart = params.HELM_CHART_NAME
                    def targetNamespace = params.TARGET_NAMESPACE
                    
                    echo "Deploying ${helmChart} to ${targetNamespace}"
                    sh "cd ${WORKSPACE}/openmrs/OpenmrsCore"
                    sh "helm upgrade --install ${helmChart} ${WORKSPACE}/openmrs/openmrs-core --set openmrs.imagename=${DOCKER_IMAGE_NAME}, openmrs.imagetag=${BUILD_NUMBER},mysql.MYSQL_PASSWORD=${MYSQL_PASSWORD},mysql.MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD},mysql.OMRS_ADMIN_USER_PASSWORD=${OMRS_ADMIN_USER_PASSWORD} --namespace ${targetNamespace}"
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