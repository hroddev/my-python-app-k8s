// Jenkinsfile
pipeline {
    agent {
        kubernetes {
            label 'kubernetes' // Usa la etiqueta que configuraste en tu Pod Template de Jenkins
            // Agentes Pod YAML: Contenedores para las herramientas necesarias
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins: agent
spec:
  containers:
  - name: docker-agent
    image: docker:dind # Imagen con Docker en Docker para construir imágenes
    command: ['cat']
    tty: true
    securityContext:
      privileged: true # Necesario para dind
    volumeMounts:
      - name: docker-sock
        mountPath: /var/run/docker.sock
  - name: kubectl-agent
    image: bitnami/kubectl:latest # Imagen con kubectl para interactuar con K8s
    command: ['cat']
    tty: true
  - name: ansible-agent
    image: cytopia/ansible:latest # Imagen con Ansible
    command: ['cat']
    tty: true
  volumes:
  - name: docker-sock
    hostPath: # Usaremos el socket de Docker del host (Docker Desktop)
      path: /var/run/docker.sock
"""
        }
    }

    environment {
        DOCKER_REGISTRY = 'localhost:5000' // Tu registro local de Kind
        IMAGE_NAME = "my-python-app"
        K8S_NAMESPACE = "default" // Puedes cambiar esto si quieres un namespace dedicado
        // Si planeas usar Terraform para la aplicación, puedes definir el path aquí
        // TERRAFORM_APP_DIR = "terraform-app-manifests"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                container('docker-agent') { // Usa el contenedor docker-agent
                    script {
                        echo "Building Docker image ${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}:${env.BUILD_NUMBER}"
                        sh "docker build -t ${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}:${env.BUILD_NUMBER} ."
                        sh "docker tag ${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}:${env.BUILD_NUMBER} ${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                container('docker-agent') { // Usa el contenedor docker-agent
                    script {
                        echo "Pushing Docker image ${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}:${env.BUILD_NUMBER}"
                        sh "docker push ${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}:${env.BUILD_NUMBER}"
                        sh "docker push ${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl-agent') { // Usa el contenedor kubectl-agent
                    script {
                        echo "Applying Kubernetes deployment..."
                        // ¡Aquí es donde leemos el archivo YAML!
                        sh "kubectl apply -f k8s-deployment.yaml -n ${env.K8S_NAMESPACE}"
                    }
                }
            }
        }

        // Opcional: Integración con Ansible (si necesitas tareas de configuración más complejas)
        stage('Run Ansible Playbook (Optional)') {
            steps {
                container('ansible-agent') { // Usa el contenedor ansible-agent
                    script {
                        echo "Running Ansible playbook (example only)..."
                        // Aquí iría tu playbook Ansible. Por ejemplo, para verificar despliegue.
                        // sh "ansible-playbook -i localhost, your_ansible_playbook.yaml"
                        sh "echo 'Ansible integration placeholder. No playbook executed.'"
                    }
                }
            }
        }

        // Opcional: Integración con Terraform para manejar manifiestos de la aplicación
        // Si quisieras que Terraform gestione k8s-deployment.yaml, podrías hacer algo así:
        /*
        stage('Deploy with Terraform (Optional)') {
            steps {
                container('kubectl-agent') { // kubectl-agent para asegurar acceso al clúster para Terraform
                    script {
                        echo "Initializing and applying Terraform for application deployment..."
                        dir("${env.TERRAFORM_APP_DIR}") { // Asume que tienes un directorio para Terraform de la app
                            sh "terraform init"
                            sh "terraform apply -auto-approve"
                        }
                    }
                }
            }
        }
        */
    }

    post {
        always {
            echo "Pipeline finished."
        }
        success {
            echo "Pipeline completed successfully."
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}