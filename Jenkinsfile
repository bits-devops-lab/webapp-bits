pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/bits-devops-lab/webapp-bits.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t webapp-using-pipeline:latest .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker tag webapp-using-pipeline:latest mdrhlonedoto/webapp-using-pipeline:latest'
                    sh 'docker push mdrhlonedoto/webapp-using-pipeline:latest'
                }
            }
        }

        stage('Deploy to Kubernetes') {
          steps {
            withCredentials([string(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_CONTENT')]) {
             sh '''
            # Use a here-document (cat <<EOF) to safely write the multiline content.
            # This is cleaner than 'echo' for YAML.
            cat <<EOF > kubeconfig-temp-file.yaml
            $KUBECONFIG_CONTENT
            EOF
            
            # Export the KUBECONFIG variable to the path of the new file
            export KUBECONFIG=$(pwd)/kubeconfig-temp-file.yaml
            
            kubectl apply -f webapp-bits-deployment.yaml
            kubectl rollout status deployment/webapp-bits
            kubectl get svc webapp-bits-service
            '''
            }
          }
        }
    }
}
