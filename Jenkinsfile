pipeline {
    agent any

    environment {
        DOCKER_CREDS = 'Docker_reshma'
        DOCKER_REG = 'reshmanavale'
        GIT_URL = 'https://github.com/reshmanavale/Learner-Report-Application-Deployment.git'
        GIT_BRANCH = 'main'
        AWS_ACCESS_KEY_ID = 'AWS_ACCESS_KEY_ID' // AWS Access Key ID stored in Jenkins
        AWS_SECRET_ACCESS_KEY = 'AWS_SECRET_ACCESS_KEY' // AWS Secret Access Key stored in Jenkins
        AWS_REGION = 'us-west-1' // AWS region where the cluster is located
        CLUSTER_NAME = 'reshma-learnerapplication-cluster' // EKS cluster name
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Checking out code from GitHub...'
                git url: "${env.GIT_URL}", branch: "${env.GIT_BRANCH}"
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    echo 'Building Docker images...'
                    def backendImage = docker.build("${DOCKER_REG}/learnerreportbe:v1.0", "--no-cache learnerReportCS_backend")
                    def frontendImage = docker.build("${DOCKER_REG}/learnerreportfe:v1.0", "--no-cache learnerReportCS_frontend")
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                script {
                    echo 'Pushing Docker images to Docker Hub...'
                    docker.withRegistry('', DOCKER_CREDS) {
                        docker.image("${DOCKER_REG}/learnerreportbe:v1.0").push()
                        docker.image("${DOCKER_REG}/learnerreportfe:v1.0").push()
                    }
                }
            }
        }

        stage('Configure kubectl') {
            steps {
                script {
                    echo 'Configuring kubectl with AWS EKS...'
                    sh """
                    export AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
                    export AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
                    export AWS_REGION=${AWS_REGION}
                    aws --version
                    
                    """
                }
            }
        }

        stage('Create Kubernetes Cluster') {
            steps {
                script {
                    sh """
                    # Check if cluster exists
                    if ! aws eks describe-cluster --name ${CLUSTER_NAME} --query 'cluster.status' --output text 2>/dev/null; then
                        echo "Cluster does not exist. Creating a new cluster..."
                        eksctl create cluster --name ${CLUSTER_NAME} \
                            --region ${AWS_REGION} \
                            --version 1.31 \
                            --nodegroup-name reshma-standard-workers \
                            --node-type t3.medium \
                            --nodes 2 \
                            --nodes-min 1 \
                            --nodes-max 3 \
                            --with-oidc \
                            --managed
                    else
                        echo "Cluster already exists. Using existing cluster."
                    fi
                    
                    # Update kubeconfig regardless of creation
                    aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}
                    """
                }
            }
        }

         stage('Deploy Database to Kubernetes') {
            steps {
                script {
                    echo 'Deploying Backend to Kubernetes...'
                    sh """
                    kubectl apply -f './k8s/database/namespace.yml'
                    kubectl apply -f './k8s/database/db-pv.yml'
                    kubectl apply -f './k8s/database/db-pvc.yml'
                    kubectl apply -f './k8s/database/deployment.yml'
                    kubectl apply -f './k8s/database/service.yml'                    
                    """
                }
            }
        }

        stage('Deploy Backend to Kubernetes') {
            steps {
                script {
                    echo 'Deploying Backend to Kubernetes...'
                    sh """                    
                    kubectl apply -f './k8s/backend/namespace.yml'
                    kubectl apply -f './k8s/backend/secret.yml'
                    kubectl apply -f './k8s/backend/deployment.yml'
                    kubectl apply -f './k8s/backend/service.yml'
                    """
                }
            }
        }

        stage('Update loadbalancer IP in Frontend') {
            steps {
                script {
                    echo 'Updating load balancer IP in Frontend...'
                    
                    // Fetch the backend service hostname
                    def backendService = sh(script: "kubectl get svc be-service -n learner-report-backend -o jsonpath=\"{.status.loadBalancer.ingress[0].hostname}\"", returnStdout: true).trim()
                    
                    // Debugging: Print the backendService value
                    echo "Backend Service Hostname: ${backendService}"
                    
                    // Check if backendService is empty
                    if (!backendService) {
                        error "Failed to retrieve backend service hostname. Ensure the service 'be-service' is deployed and has a valid load balancer IP/hostname."
                    }
                    
                    // Create base64 encoded URL for secret
                    def backendUrl = "http://${backendService}:3001"
                    echo "Backend URL: ${backendUrl}" // Debugging: Print the backend URL
                    
                    def encodedUrl = sh(script: "echo -n '${backendUrl}' | base64", returnStdout: true).trim()
                    echo "Encoded Backend URL: ${encodedUrl}" // Debugging: Print the encoded URL
                    
                    // Use a temporary file to update secret.yml
                    sh """
                    temp_file=\$(mktemp)
                    echo "REACT_APP_BACKEND_URL: \\\"${encodedUrl}\\\"" > \$temp_file
                    sed -i '/REACT_APP_BACKEND_URL:/r \$temp_file' './k8s/frontend/secret.yml'
                    rm -f \$temp_file
                    """
                }
            }
        }

        stage('Deploy Frontend to Kubernetes') {
            steps {
                script {
                    echo 'Deploying Frontend to Kubernetes...'
                    sh """
                    kubectl apply -f './k8s/frontend/namespace.yml'
                    kubectl apply -f './k8s/frontend/secret.yml'
                    kubectl apply -f './k8s/frontend/deployment.yml'
                    kubectl apply -f './k8s/frontend/service.yml'
                    """
                }
            }
        }
    }
}
