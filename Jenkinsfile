pipeline { 
    agent any 
     
    environment { 
        DOCKER_IMAGE = 'maniamk/flask-app' 
        DOCKER_TAG = "${BUILD_NUMBER}" 
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds' 
        GITHUB_CREDENTIALS = 'github-creds' 
    } 
     
    stages { 
        // STAGE 1: CODE FETCH 
        stage('📦 Code Fetch') { 
            steps { 
                echo '===========================================' 
                echo 'STAGE 1: Fetching code from GitHub...' 
                echo '===========================================' 
                git branch: 'master', 
                    credentialsId: "${GITHUB_CREDENTIALS}", 
                    url: 'https://github.com/manahil30/devops-flask-app.git' 
                echo '✅ Code fetched successfully from GitHub!' 
                sh 'ls -la' 
            } 
        } 
         
        // STAGE 2: DOCKER IMAGE CREATION 
        stage('🐳 Docker Image Creation') { 
            steps { 
                echo '===========================================' 
                echo 'STAGE 2: Building Docker image...' 
                echo '===========================================' 
                sh """ 
                    docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} . 
                    docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest 
                """ 
                echo '✅ Docker image built successfully!' 
                 
                echo 'Pushing image to DockerHub...' 
                withCredentials([usernamePassword( 
                    credentialsId: "${DOCKERHUB_CREDENTIALS}", 
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS' 
                )]) { 
                    sh """ 
                        echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin 
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG} 
                        docker push ${DOCKER_IMAGE}:latest 
                    """ 
                } 
                echo '✅ Image pushed to DockerHub successfully!' 
            } 
        } 
         
        // STAGE 3: KUBERNETES DEPLOYMENT 
        stage('☸️ Kubernetes Deployment') { 
            steps { 
                echo '===========================================' 
                echo 'STAGE 3: Deploying to Kubernetes...' 
                echo '===========================================' 
                sh """ 
                    # Update image tag in deployment file 
                    sed -i 's|image:.*|image: ${DOCKER_IMAGE}:${DOCKER_TAG}|g' k8s/deployment.yaml 
                     
                    # Apply deployment and service 
                    kubectl apply -f k8s/deployment.yaml 
                    kubectl apply -f k8s/service.yaml 
                     
                    # Wait for rollout to complete 
                    kubectl rollout status deployment/flask-app --timeout=120s 
                     
                    # Get pod status 
                    kubectl get pods 
                """ 
                echo '✅ Kubernetes deployment successful!' 
            } 
        } 
         
        // STAGE 4: PROMETHEUS/GRAFANA SETUP 
        stage('📊 Prometheus/Grafana Setup') { 
            steps { 
                echo '===========================================' 
                echo 'STAGE 4: Setting up monitoring stack...' 
                echo '===========================================' 
                sh """ 
                    # Add Helm repository 
                    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts 
                    helm repo update 
                     
                    # Install kube-prometheus-stack 
                    helm upgrade --install prometheus prometheus-community/kube-prometheus-stack \ 
                        --namespace monitoring --create-namespace \ 
                        --set grafana.service.type=NodePort \ 
                        --set grafana.service.nodePort=30001 \ 
                        --set prometheus.service.type=NodePort \ 
                        --set prometheus.service.nodePort=30002 
                     
                    # Wait for all pods to be ready 
                    kubectl wait --namespace monitoring --for=condition=ready pod --all --timeout=180s 
                """ 
                echo '✅ Monitoring stack deployed successfully!' 
            } 
        } 
    } 
     
    post { 
        success { 
            echo '===========================================' 
            echo '🎉 PIPELINE COMPLETED SUCCESSFULLY! 🎉' 
            echo '===========================================' 
            echo "Application URL: http://<ec2-ip>:30000" 
            echo "Grafana URL: http://<ec2-ip>:30001 (Username: admin, Password: prom-operator)" 
            echo "Prometheus URL: http://<ec2-ip>:30002" 
        } 
        failure { 
            echo '===========================================' 
            echo '❌ PIPELINE FAILED!' 
            echo '===========================================' 
            echo 'Check the logs above for error details.' 
        } 
    } 
}
