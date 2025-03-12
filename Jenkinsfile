pipeline {
    agent any

    environment {
        // Nombre de la imagen Docker
        DOCKER_IMAGE_NAME = "turepo/mundosE-pipelineProyect"
        // Dirección IP del servidor donde se desplegará la aplicación
        DEPLOY_SERVER = "192.168.0.5"
        // Usuario con permisos en el servidor
        DEPLOY_USER = "tuusuario"
        // Directorio en el servidor donde se ejecutará la aplicación
        APP_DIR = "/home/tuusuario/node-app"
    }

    stages {
        stage('Checkout') {
            steps {
                // Clona el repositorio de GitHub en la rama "development"
                git branch: 'development', url: 'https://github.com/TU_USUARIO/TU_REPO.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Construye la imagen Docker con el tag basado en el número de build de Jenkins
                    sh "docker build -t ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} ."
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    // Ejecuta los tests dentro del contenedor y luego lo elimina
                    sh "docker run --rm ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} npm test"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Autentica en Docker Hub con las credenciales almacenadas en Jenkins
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                        // Login en Docker Hub
                        sh "echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_HUB_USERNAME} --password-stdin"
                        // Sube la imagen con el tag del número de build
                        sh "docker push ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                        // Marca la imagen como "latest" y la vuelve a subir
                        sh "docker tag ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} ${DOCKER_IMAGE_NAME}:latest"
                        sh "docker push ${DOCKER_IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                script {
                    // Se conecta al servidor vía SSH y actualiza el contenedor con la nueva imagen
                    sshagent(['ssh-credentials']) {
                        sh """
                        ssh ${DEPLOY_USER}@${DEPLOY_SERVER} <<EOF
                        # Descargar la última versión de la imagen
                        docker pull ${DOCKER_IMAGE_NAME}:latest
                        # Detener el contenedor anterior si está corriendo
                        docker stop node-app || true
                        # Eliminar el contenedor antiguo
                        docker rm node-app || true
                        # Ejecutar el nuevo contenedor en segundo plano
                        docker run -d --name node-app -p 3000:3000 ${DOCKER_IMAGE_NAME}:latest
                        EOF
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                try {
                    // Limpia la imagen de Docker en la máquina de Jenkins para liberar espacio
                    sh "docker rmi ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                } catch (Exception e) {
                    echo 'No se pudo eliminar la imagen local de Docker.'
                }
            }
        }
    }
}
